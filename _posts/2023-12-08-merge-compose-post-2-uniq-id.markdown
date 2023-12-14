---
layout: post
title:  "2023-12-08 Merge UniqID+ComposePost"
date:   2023-12-08 1:53:46 -0500
categories: serverless functions
---
### Call graph of socialNetwork

![socialNetwork](/assets/2023-12-08/socialNet_arch.png)

### Steps to merge to serverless functions

![diagram](/assets/2023-12-08/diagram.png)

- an example

```bash
# compile the code into LLVM IR
clang -I$NIGHTCORE_PATH/include -fPIC -emit-llvm -S $NIGHTCORE_PATH/examples/c_merge/foo.c -c -o foo.ll
clang -I$NIGHTCORE_PATH/include -fPIC -emit-llvm -S $NIGHTCORE_PATH/examples/c_merge/bar.c -c -o bar.ll

# change function name "faas_func_call" in bar to be "faas_func_callee"
# also delete "faas_init", "faas_create_func_worker" and "faas_destroy_func_worker"
opt -load $LLVM_PATH/build/lib/LLVMMergeFunc.so -enable-new-pm=0 -ChangeFuncName bar.ll -S -o bar_only.ll

# merge "faas_func_callee" into foo's address space
llvm-link foo.ll bar_only.ll -o foo_rpc_bar.ll -S

# change the "faas_func_callee" to be "Bar" (normal function call)
opt -load $LLVM_PATH/build/lib/LLVMMergeFunc.so -S -enable-new-pm=0 -o foo_bar.ll -MergeFunc < foo_rpc_bar.ll

# finally generate obj of libfoo and link the .obj file
llc -filetype=obj -relocation-model=pic foo_bar.ll -o libfoo.o
clang -shared -fPIC -O2 -I../../include libfoo.o -o libfoo.so 
```

### Duplication of functions in Caller and Callee address space
- a list of functions symbols that occur in both `libUniqueIdService.so` and `libComposePostService.so`
	+ there are <strong>6,395</strong> functions in `UniqueIdService` in total
- <strong>Category 1: can be removed</strong>
	+ all the functions are from some header files. 
	+ same function name and same function body
	+ duplicated function can be removed

| mangled function symbol  | location | 
| :----: | :----:|
| _ZN14social_network11init_loggerEv | src/logger.h |
| _ZN14social_network16load_config_fileERKNSt7__cxx1112basic_string<br>IcSt11char_traitsIcESaIcEEEPN8nlohmann10basic_jsonISt3mapSt6vector<br>S5_blmdSaNS8_14adl_serializerEEE | src/utils.h | 
| _ZN14social_network11load_configEPN8nlohmann10basic_jsonISt3mapSt6<br>vectorNSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEEEblmdSaNS<br>0_14adl_serializerEEE | src/logger.h|
| _ZN14social_network11SetUpTracerERKNSt7__cxx1112basic_stringIcSt11char<br>_traitsIcESaIcEEES7_ | src/tracing.h |

- <strong>Category 2: cannot be removed</strong>
	+ same function name but different function body

| function symbol  | caller location | callee location |
| :----: | :----:| :----:|
| faas_init | src/UniqueIdService/UniqueIdService.cpp | src/ComposePostService/ComposePostService.cpp |
| faas_create_func_worker | src/UniqueIdService/UniqueIdService.cpp |src/ComposePostService/ComposePostService.cpp |
| faas_destroy_func_worker | src/UniqueIdService/UniqueIdService.cpp |src/ComposePostService/ComposePostService.cpp |
| faas_func_call | src/UniqueIdService/UniqueIdService.cpp |src/ComposePostService/ComposePostService.cpp |


### Remove redundant functions 

![rename](/assets/2023-12-08/rename.png)

- How to decide which functions should be removed? 
	+ check the file name of the function
		* all functions from `src/logger.h`, `src/tracing.h`, `src/utils.h` have a possibility to have a duplicated version in callee.
		* mark all functions from `src/logger.h`, `src/tracing.h`, `src/utils.h`
	+ How to check the source file name of a function at IR level
		* LLVM's general debug info `-g`
		* some useful tutorial for accessing LLVM's general debug info at IR level
			* [here](https://github.com/zyuxuan0115/misc-2023-11-20/blob/main/dmon/llvm-passes/selective-prefetch/Prefetch.cpp#L254) is how DMon read debug info from LLVM's optimization pass
			* [How to strip off debug info in llvm IR](https://discourse.llvm.org/t/how-to-strip-off-metadata-debugging-information/70201)

- Instead remove the redundant functions, annotate them
	+ by changing their function name to be `<original_func_name>_redundant`
	+ why not delete them?
		* because they might have caller functions

![delete](/assets/2023-12-08/remove.png)

- when deleting a function, the call instruction in the corresponding caller function should also be changed
	+ From [how to get caller function of a function](https://discourse.llvm.org/t/how-to-get-the-caller-functions-of-a-function-in-llvm/61830), we can build a call graph
	+ [an example](https://stackoverflow.com/questions/45073555/llvm-callgraphnode-giving-incorrect-function-name) of building a call graph		
	+ [LLVM call graph class](https://llvm.org/doxygen/classllvm_1_1CallGraphNode.html) 

![call](/assets/2023-12-08/call.png)


### Merge Serverless functions
- There are 3 functions that needs to be merged: `faas_init`, `faas_create_func_worker`, `faas_destroy_func_worker`
- There is 1 function that acts as both the caller and the callee: `faas_func_call`
	+ e.g. in `foo`'s address space `faas_func_call` calls another `faas_func_call` in `bar`'s address space


#### (1) 3 functions that needs to be merged

- both serverless functions (caller: `UniqID`, callee: `composePost`) have the same `faas_destroy_func_worker`
  + finally we will only have 1 `faas_worker` for the merged serverless function
	+ one of the `faas_destroy_func_worker`s can be deleted. (we don't need to free the `faas_worker` twice)

```c++
int faas_destroy_func_worker(void* worker_handle) {
    FaasWorker* faas_worker = reinterpret_cast<FaasWorker*>(worker_handle);
    delete faas_worker;
    return 0;
}
```

- 


- the next thing is to add if statement before the invocation of the serverless function
 
- the function name of callee is: `ComposePostService`
	+ I got it from [here](https://github.com/ut-osa/nightcore-benchmarks/blob/master/workloads/DeathStarBench/socialNetwork/src/UniqueIdService/UniqueIdService.cpp#L52)

```llvm
; Function Attrs: mustprogress noinline nounwind optnone uwtable
define linkonce_odr void @_ZN10FaasWorker12SetProcessorESt10shared_ptrIN6apache6thrift10TProcessorEE(%class.FaasWorker* nonnull align 8 dereferenceable(120) %0, %"class.std::shared_ptr.81"* %1) #4 comdat align 2 {
  %3 = alloca %class.FaasWorker*, align 8
  store %class.FaasWorker* %0, %class.FaasWorker** %3, align 8
  %4 = load %class.FaasWorker*, %class.FaasWorker** %3, align 8
  %5 = getelementptr inbounds %class.FaasWorker, %class.FaasWorker* %4, i32 0, i32 3
  %6 = call nonnull align 8 dereferenceable(16) %"class.std::shared_ptr.81"* @_ZNSt10shared_ptrIN6apache6thrift10TProcessorEEaSERKS3_(%"class.std::shared_ptr.81"* nonnull align 8 dereferenceable(16) %5, %"class.std::shared_ptr.81"* nonnull align 8 dereferenceable(16) %1) #21
  ret void
}
```


```llvm
; Function Attrs: mustprogress noinline optnone uwtable
define linkonce_odr zeroext i1 @_ZN10FaasWorker7ProcessEPKcm(%class.FaasWorker* nonnull align 8 dereferenceable(120) %0, i8* %1, i64 %2) #1 comdat align 2 personality i8* bitcast (i32 (...)* @__gxx_personality_v0 to i8*) {
  %4 = alloca i1, align 1
  %5 = alloca %class.FaasWorker*, align 8
  %6 = alloca i8*, align 8
  %7 = alloca i64, align 8
  %8 = alloca %"class.std::shared_ptr.70", align 8
  %9 = alloca %"class.std::shared_ptr.73", align 8
  %10 = alloca i8*, align 8
  %11 = alloca i32, align 4
  %12 = alloca %"class.std::shared_ptr.70", align 8
  %13 = alloca %"class.std::shared_ptr.73", align 8
  %14 = alloca %"class.social_network::UniqueIdServiceIf"*, align 8
  store %class.FaasWorker* %0, %class.FaasWorker** %5, align 8
  store i8* %1, i8** %6, align 8
  store i64 %2, i64* %7, align 8
  %15 = load %class.FaasWorker*, %class.FaasWorker** %5, align 8
  %16 = getelementptr inbounds %class.FaasWorker, %class.FaasWorker* %15, i32 0, i32 3
  %17 = call zeroext i1 @_ZSteqIN6apache6thrift10TProcessorEEbRKSt10shared_ptrIT_EDn(%"class.std::shared_ptr.81"* nonnull align 8 dereferenceable(16) %16, i8* null) #21
  br i1 %17, label %18, label %21

18:                                               ; preds = %3
  %19 = load %struct._IO_FILE*, %struct._IO_FILE** @stderr, align 8
  %20 = call i32 (%struct._IO_FILE*, i8*, ...) @fprintf(%struct._IO_FILE* %19, i8* getelementptr inbounds ([23 x i8], [23 x i8]* @.str.169, i64 0, i64 0))
  store i1 false, i1* %4, align 1
  br label %95

21:                                               ; preds = %3
  %22 = getelementptr inbounds %class.FaasWorker, %class.FaasWorker* %15, i32 0, i32 4
  %23 = bitcast %"class.std::shared_ptr.87"* %22 to %"struct.std::less.113"*
  %24 = call %"class.apache::thrift::transport::TMemoryBuffer"* @_ZNKSt19__shared_ptr_accessIN6apache6thrift9transport13TMemoryBufferELN9__gnu_cxx12_Lock_policyE2ELb0ELb0EEptEv(%"struct.std::less.113"* nonnull align 1 dereferenceable(1) %23) #21
  %25 = load i8*, i8** %6, align 8
  %26 = load i64, i64* %7, align 8
  %27 = trunc i64 %26 to i32
  call void @_ZN6apache6thrift9transport13TMemoryBuffer11resetBufferEPhjNS2_12MemoryPolicyE(%"class.apache::thrift::transport::TMemoryBuffer"* nonnull align 8 dereferenceable(57) %24, i8* %25, i32 %27, i32 1)
  %28 = getelementptr inbounds %class.FaasWorker, %class.FaasWorker* %15, i32 0, i32 3
  %29 = bitcast %"class.std::shared_ptr.81"* %28 to %"struct.std::less.113"*
  %30 = call %"class.apache::thrift::TProcessor"* @_ZNKSt19__shared_ptr_accessIN6apache6thrift10TProcessorELN9__gnu_cxx12_Lock_policyE2ELb0ELb0EEptEv(%"struct.std::less.113"* nonnull align 1 dereferenceable(1) %29) #21
  %31 = getelementptr inbounds %class.FaasWorker, %class.FaasWorker* %15, i32 0, i32 6
  %32 = bitcast %"class.std::shared_ptr.73"* %31 to %"struct.std::less.113"*
  %33 = call %"class.social_network::UniqueIdServiceIf"* @_ZNKSt19__shared_ptr_accessIN6apache6thrift8protocol16TProtocolFactoryELN9__gnu_cxx12_Lock_policyE2ELb0ELb0EEptEv(%"struct.std::less.113"* nonnull align 1 dereferenceable(1) %32) #21
  %34 = getelementptr inbounds %class.FaasWorker, %class.FaasWorker* %15, i32 0, i32 4
  call void @_ZNSt10shared_ptrIN6apache6thrift9transport10TTransportEEC2INS2_13TMemoryBufferEvEERKS_IT_E(%"class.std::shared_ptr.73"* nonnull align 8 dereferenceable(16) %9, %"class.std::shared_ptr.87"* nonnull align 8 dereferenceable(16) %34) #21
  %35 = bitcast %"class.social_network::UniqueIdServiceIf"* %33 to void (%"class.std::shared_ptr.70"*, %"class.social_network::UniqueIdServiceIf"*, %"class.std::shared_ptr.73"*)***
  %36 = load void (%"class.std::shared_ptr.70"*, %"class.social_network::UniqueIdServiceIf"*, %"class.std::shared_ptr.73"*)**, void (%"class.std::shared_ptr.70"*, %"class.social_network::UniqueIdServiceIf"*, %"class.std::shared_ptr.73"*)*** %35, align 8
  %37 = getelementptr inbounds void (%"class.std::shared_ptr.70"*, %"class.social_network::UniqueIdServiceIf"*, %"class.std::shared_ptr.73"*)*, void (%"class.std::shared_ptr.70"*, %"class.social_network::UniqueIdServiceIf"*, %"class.std::shared_ptr.73"*)** %36, i64 2
  %38 = load void (%"class.std::shared_ptr.70"*, %"class.social_network::UniqueIdServiceIf"*, %"class.std::shared_ptr.73"*)*, void (%"class.std::shared_ptr.70"*, %"class.social_network::UniqueIdServiceIf"*, %"class.std::shared_ptr.73"*)** %37, align 8
  invoke void %38(%"class.std::shared_ptr.70"* sret(%"class.std::shared_ptr.70") align 8 %8, %"class.social_network::UniqueIdServiceIf"* nonnull align 8 dereferenceable(8) %33, %"class.std::shared_ptr.73"* %9)
          to label %39 unwind label %57

39:                                               ; preds = %21
  %40 = getelementptr inbounds %class.FaasWorker, %class.FaasWorker* %15, i32 0, i32 7
  %41 = bitcast %"class.std::shared_ptr.73"* %40 to %"struct.std::less.113"*
  %42 = call %"class.social_network::UniqueIdServiceIf"* @_ZNKSt19__shared_ptr_accessIN6apache6thrift8protocol16TProtocolFactoryELN9__gnu_cxx12_Lock_policyE2ELb0ELb0EEptEv(%"struct.std::less.113"* nonnull align 1 dereferenceable(1) %41) #21
  %43 = getelementptr inbounds %class.FaasWorker, %class.FaasWorker* %15, i32 0, i32 5
  call void @_ZNSt10shared_ptrIN6apache6thrift9transport10TTransportEEC2ERKS4_(%"class.std::shared_ptr.73"* nonnull align 8 dereferenceable(16) %13, %"class.std::shared_ptr.73"* nonnull align 8 dereferenceable(16) %43) #21
  %44 = bitcast %"class.social_network::UniqueIdServiceIf"* %42 to void (%"class.std::shared_ptr.70"*, %"class.social_network::UniqueIdServiceIf"*, %"class.std::shared_ptr.73"*)***
  %45 = load void (%"class.std::shared_ptr.70"*, %"class.social_network::UniqueIdServiceIf"*, %"class.std::shared_ptr.73"*)**, void (%"class.std::shared_ptr.70"*, %"class.social_network::UniqueIdServiceIf"*, %"class.std::shared_ptr.73"*)*** %44, align 8
  %46 = getelementptr inbounds void (%"class.std::shared_ptr.70"*, %"class.social_network::UniqueIdServiceIf"*, %"class.std::shared_ptr.73"*)*, void (%"class.std::shared_ptr.70"*, %"class.social_network::UniqueIdServiceIf"*, %"class.std::shared_ptr.73"*)** %45, i64 2
  %47 = load void (%"class.std::shared_ptr.70"*, %"class.social_network::UniqueIdServiceIf"*, %"class.std::shared_ptr.73"*)*, void (%"class.std::shared_ptr.70"*, %"class.social_network::UniqueIdServiceIf"*, %"class.std::shared_ptr.73"*)** %46, align 8
  invoke void %47(%"class.std::shared_ptr.70"* sret(%"class.std::shared_ptr.70") align 8 %12, %"class.social_network::UniqueIdServiceIf"* nonnull align 8 dereferenceable(8) %42, %"class.std::shared_ptr.73"* %13)
          to label %48 unwind label %61

48:                                               ; preds = %39
  %49 = bitcast %"class.apache::thrift::TProcessor"* %30 to i1 (%"class.apache::thrift::TProcessor"*, %"class.std::shared_ptr.70"*, %"class.std::shared_ptr.70"*, i8*)***
  %50 = load i1 (%"class.apache::thrift::TProcessor"*, %"class.std::shared_ptr.70"*, %"class.std::shared_ptr.70"*, i8*)**, i1 (%"class.apache::thrift::TProcessor"*, %"class.std::shared_ptr.70"*, %"class.std::shared_ptr.70"*, i8*)*** %49, align 8
  %51 = getelementptr inbounds i1 (%"class.apache::thrift::TProcessor"*, %"class.std::shared_ptr.70"*, %"class.std::shared_ptr.70"*, i8*)*, i1 (%"class.apache::thrift::TProcessor"*, %"class.std::shared_ptr.70"*, %"class.std::shared_ptr.70"*, i8*)** %50, i64 2
  %52 = load i1 (%"class.apache::thrift::TProcessor"*, %"class.std::shared_ptr.70"*, %"class.std::shared_ptr.70"*, i8*)*, i1 (%"class.apache::thrift::TProcessor"*, %"class.std::shared_ptr.70"*, %"class.std::shared_ptr.70"*, i8*)** %51, align 8
  %53 = invoke zeroext i1 %52(%"class.apache::thrift::TProcessor"* nonnull align 8 dereferenceable(24) %30, %"class.std::shared_ptr.70"* %8, %"class.std::shared_ptr.70"* %12, i8* null)
          to label %54 unwind label %65

54:                                               ; preds = %48
  %55 = xor i1 %53, true
  call void @_ZNSt10shared_ptrIN6apache6thrift8protocol9TProtocolEED2Ev(%"class.std::shared_ptr.70"* nonnull align 8 dereferenceable(16) %12) #21
  call void @_ZNSt10shared_ptrIN6apache6thrift9transport10TTransportEED2Ev(%"class.std::shared_ptr.73"* nonnull align 8 dereferenceable(16) %13) #21
  call void @_ZNSt10shared_ptrIN6apache6thrift8protocol9TProtocolEED2Ev(%"class.std::shared_ptr.70"* nonnull align 8 dereferenceable(16) %8) #21
  call void @_ZNSt10shared_ptrIN6apache6thrift9transport10TTransportEED2Ev(%"class.std::shared_ptr.73"* nonnull align 8 dereferenceable(16) %9) #21
  br i1 %55, label %56, label %88

56:                                               ; preds = %54
  store i1 false, i1* %4, align 1
  br label %95

57:                                               ; preds = %21
  %58 = landingpad { i8*, i32 }
          cleanup
          catch i8* bitcast (i8** @_ZTISt9exception to i8*)
  %59 = extractvalue { i8*, i32 } %58, 0
  store i8* %59, i8** %10, align 8
  %60 = extractvalue { i8*, i32 } %58, 1
  store i32 %60, i32* %11, align 4
  br label %70

61:                                               ; preds = %39
  %62 = landingpad { i8*, i32 }
          cleanup
          catch i8* bitcast (i8** @_ZTISt9exception to i8*)
  %63 = extractvalue { i8*, i32 } %62, 0
  store i8* %63, i8** %10, align 8
  %64 = extractvalue { i8*, i32 } %62, 1
  store i32 %64, i32* %11, align 4
  br label %69

65:                                               ; preds = %48
  %66 = landingpad { i8*, i32 }
          cleanup
          catch i8* bitcast (i8** @_ZTISt9exception to i8*)
  %67 = extractvalue { i8*, i32 } %66, 0
  store i8* %67, i8** %10, align 8
  %68 = extractvalue { i8*, i32 } %66, 1
  store i32 %68, i32* %11, align 4
  call void @_ZNSt10shared_ptrIN6apache6thrift8protocol9TProtocolEED2Ev(%"class.std::shared_ptr.70"* nonnull align 8 dereferenceable(16) %12) #21
  br label %69

69:                                               ; preds = %65, %61
  call void @_ZNSt10shared_ptrIN6apache6thrift9transport10TTransportEED2Ev(%"class.std::shared_ptr.73"* nonnull align 8 dereferenceable(16) %13) #21
  call void @_ZNSt10shared_ptrIN6apache6thrift8protocol9TProtocolEED2Ev(%"class.std::shared_ptr.70"* nonnull align 8 dereferenceable(16) %8) #21
  br label %70

70:                                               ; preds = %69, %57
  call void @_ZNSt10shared_ptrIN6apache6thrift9transport10TTransportEED2Ev(%"class.std::shared_ptr.73"* nonnull align 8 dereferenceable(16) %9) #21
  br label %71

71:                                               ; preds = %70
  %72 = load i32, i32* %11, align 4
  %73 = call i32 @llvm.eh.typeid.for(i8* bitcast (i8** @_ZTISt9exception to i8*)) #21
  %74 = icmp eq i32 %72, %73
  br i1 %74, label %75, label %97

75:                                               ; preds = %71
  %76 = load i8*, i8** %10, align 8
  %77 = call i8* @__cxa_begin_catch(i8* %76) #21
  %78 = bitcast i8* %77 to %"class.social_network::UniqueIdServiceIf"*
  store %"class.social_network::UniqueIdServiceIf"* %78, %"class.social_network::UniqueIdServiceIf"** %14, align 8
  %79 = load %struct._IO_FILE*, %struct._IO_FILE** @stderr, align 8
  %80 = load %"class.social_network::UniqueIdServiceIf"*, %"class.social_network::UniqueIdServiceIf"** %14, align 8
  %81 = bitcast %"class.social_network::UniqueIdServiceIf"* %80 to i8* (%"class.social_network::UniqueIdServiceIf"*)***
  %82 = load i8* (%"class.social_network::UniqueIdServiceIf"*)**, i8* (%"class.social_network::UniqueIdServiceIf"*)*** %81, align 8
  %83 = getelementptr inbounds i8* (%"class.social_network::UniqueIdServiceIf"*)*, i8* (%"class.social_network::UniqueIdServiceIf"*)** %82, i64 2
  %84 = load i8* (%"class.social_network::UniqueIdServiceIf"*)*, i8* (%"class.social_network::UniqueIdServiceIf"*)** %83, align 8
  %85 = call i8* %84(%"class.social_network::UniqueIdServiceIf"* nonnull align 8 dereferenceable(8) %80) #21
  %86 = invoke i32 (%struct._IO_FILE*, i8*, ...) @fprintf(%struct._IO_FILE* %79, i8* getelementptr inbounds ([31 x i8], [31 x i8]* @.str.170, i64 0, i64 0), i8* %85)
          to label %87 unwind label %89

87:                                               ; preds = %75
  store i1 false, i1* %4, align 1
  call void @__cxa_end_catch()
  br label %95

88:                                               ; preds = %54
  br label %94

89:                                               ; preds = %75
  %90 = landingpad { i8*, i32 }
          cleanup
  %91 = extractvalue { i8*, i32 } %90, 0
  store i8* %91, i8** %10, align 8
  %92 = extractvalue { i8*, i32 } %90, 1
  store i32 %92, i32* %11, align 4
  invoke void @__cxa_end_catch()
          to label %93 unwind label %102

93:                                               ; preds = %89
  br label %97

94:                                               ; preds = %88
  store i1 true, i1* %4, align 1
  br label %95

95:                                               ; preds = %94, %87, %56, %18
  %96 = load i1, i1* %4, align 1
  ret i1 %96

97:                                               ; preds = %93, %71
  %98 = load i8*, i8** %10, align 8
  %99 = load i32, i32* %11, align 4
  %100 = insertvalue { i8*, i32 } undef, i8* %98, 0
  %101 = insertvalue { i8*, i32 } %100, i32 %99, 1
  resume { i8*, i32 } %101

102:                                              ; preds = %89
  %103 = landingpad { i8*, i32 }
          catch i8* null
  %104 = extractvalue { i8*, i32 } %103, 0
  call void @__clang_call_terminate(i8* %104) #28
  unreachable
}
```


- in BasicBlock 48, the following `invoke` instruction is the [processor_->process()](https://github.com/zyuxuan0115/nightcore-test/blob/main/socialnetwork_singlenode/DeathStarBench/socialNetwork/src/FaasWorker.h#L43) function call

```
%53 = invoke zeroext i1 %52(%"class.apache::thrift::TProcessor"* nonnull align 8 dereferenceable(24) %30, %"class.std::shared_ptr.70"* %8, %"class.std::shared_ptr.70"* %12, i8 * null)
```

### Some useful LLVM tutorial (might be useful later)
- things about [GetElementPtr](https://llvm.org/docs/GetElementPtr.html) 
- [Insert int variable into the class](https://stackoverflow.com/questions/36439248/insert-int-variable-into-the-class-using-llvm-pass-or-clang)
	+ [AST Matcher](https://clang.llvm.org/docs/LibASTMatchersReference.html)

---
layout: post
title:  "2023-11-18 nightcore & llvm"
date:   2023-11-18 1:53:46 -0500
categories: serverless functions
---
### more thoughts about pkey
- build a library just like libmpk
	+ a map maintains the relation between virtual key and physical key (1,2,3,...,15)
	+ `physical key -> virtual key`: distributes the 16 keys to a larger space.
		* harder for attackers to figure out the value of the virtual keys
	+ create a bunch of APIs that use the virtual key as the argument but have the same semantics as the original pkey calls (e.g., `pkey_alloc`, `pkey_mprotect`)
		* these APIs should be based on the original pkey calls. 
- have another `ld_preload library` that intercepts the calls of the original `pkey_set()`, `pkey_alloc()`, `pkey_mprotect()` etc. that are directly executed by the user code.
	+ the technique is called [LD_PRELOAD trick](https://stackoverflow.com/questions/426230/what-is-the-ld-preload-trick)
	+ once a pkey call being executed from user code is detected, we disable it
	+ how do we distinguish whether a pkey call is from our new library code or from user code?
		* using `libunwind` to get the function caller info from the call stack.

### understand how nightcore's external call works

### measure vhive’s function invocation time

### LLVM pass identifies the arguments of a RPC

- setup LLVM ([llvm-10.0 source code (tar.gz)](https://github.com/llvm/llvm-project/releases/tag/llvmorg-10.0.0))

```bash
> git clone https://github.com/llvm/llvm-project.git
> cd llvm-project
> mkdir build && cd build
> cmake -G "Unix Makefiles" -DCMAKE_BUILD_TYPE="Release" -DLLVM_ENABLE_PROJECTS="clang;clang-tools-extra;compiler-rt;lldb;lld" DLLVM_ENABLE_RUNTIMES="libcxx;libcxxabi" ../llvm
> make -j
```

- build passes
link from UTAustin [here](https://www.cs.utexas.edu/~pingali/CS380C/2020/assignments/llvm-guide.html)

```bash
> cp -r /proj/zyuxuanssf-PG0/nightcore-test/MergeFunc /proj/zyuxuanssf-PG0/llvm-project/llvm/lib/Transforms/
> echo 'add_subdirectory(MergeFunc)' >> /proj/zyuxuanssf-PG0/llvm-project/llvm/lib/CMakeList.txt
> cd llvm-project/build && make -j
```

- in `nightcore/examples/c`
  + `-g` adds general debug info to IR. might be useful for identify the virtual function call
  
```bash
> clang -I/proj/zyuxuanssf-PG0/nightcore/include -g -emit-llvm -S foo.c
> opt -load /proj/zyuxuanssf-PG0/llvm-project/build/lib/LLVMMergeFunc.so -MergeFunc < foo.ll > /dev/null
```

- how to identify the `faas_func_call` function
	+ some docs that might be useful in the future
		* [source level debugging](https://llvm.org/docs/SourceLevelDebugging.html): LLVM debug information always provides information to accurately read the source-level state of the program.
		* [llvm.dbg.value](https://llvm.org/docs/SourceLevelDebugging.html#llvm-dbg-value)	
		* [debug info assignment tracking](https://llvm.org/docs/AssignmentTracking.html): This intrinsic marks the position in IR where a source assignment occurred. It encodes the value of the variable.
	+ why do we identify serverless function invocation in this way?
		* because the format of function call invocation is fixed. 
		* see [the nightcore code](https://github.com/ut-osa/nightcore/blob/asplos-release/include/faas/worker_v1_interface.h#L22)

```c++
typedef int (*faas_invoke_func_fn_t)(
    void* caller_context, const char* func_name,
    const char* input_data, size_t input_length,
    const char** output_data, size_t* output_length);
```

- useful things
	+ [LLVM: print value of global variable char array?](https://stackoverflow.com/questions/53960500/llvm-how-to-convert-constantexpr-to-constantdataarray-so-i-can-print-value-of)
	+ [Get global string value in LLVM](https://stackoverflow.com/questions/50818343/get-global-string-value-in-llvm)

### LLVM merge functions from different files into one bitcode
- merge 2 bitcode into 1 bitcode

```bash
> clang -emit-llvm foo.c -c -o foo.bc
> clang -emit-llvm bar.c -c -o bar.bc
> llvm-link foo.bc bar.bc -o newfoo.ll -S
```

- one problem
	+ `foo` and `bar` used the same functions API provided by nightcore
		* `faas_init()`, `faas_func_call()`, `faas_create_func_worker()`, `faas_destroy_func_worker()`
	+ solution - a new LLVM pass that
		* changes the name of `faas_func_call()` since it's the real `Bar`
		* removes the other functions (`faas_init()`, `faas_create_func_worker()`, `faas_destroy_func_worker()`)
	+ So the final version is like this:

```bash
> clang -I/proj/zyuxuanssf-PG0/nightcore/include -g -emit-llvm foo.c -c -o foo.bc
> clang -I/proj/zyuxuanssf-PG0/nightcore/include -g -emit-llvm bar.c -c -o bar.bc
> opt -load /proj/zyuxuanssf-PG0/llvm-project/build/lib/LLVMMergeFunc.so -ChangeFuncName bar.bc -o bar_func_only.bc
> llvm-link foo.bc bar_func_only.bc -o newfoo.ll -S
> opt -load /proj/zyuxuanssf-PG0/llvm-project/build/lib/LLVMMergeFunc.so -MergeFunc < newfoo.ll > /dev/null
```

### LLVM pass converts PRC into normal calls
- [LLVM: how to create a call instruction](https://llvm.org/doxygen/classllvm_1_1CallInst.html)
- [create new function in LLVM](https://stackoverflow.com/questions/17297109/create-new-function-in-llvm)
- [my own code for creating functions to llvm](https://github.com/zyuxuan0115/cis573/blob/main/cis573lab11/src/Instrument.cpp)

### faster inter-thread synchronization


### misc
- [vtable](https://llvm.org/devmtg/2021-11/slides/2021-RelativeVTablesinC.pdf) again 

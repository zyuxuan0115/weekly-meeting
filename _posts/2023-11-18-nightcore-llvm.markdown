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

### compiler identifies the arguments of a RPC
- [how to get the argument of a function](https://stackoverflow.com/questions/9225315/find-arguments-of-a-function-in-llvm-ir)
	+ this does not work when the function is a function pointer
	+ [here](https://llvm.org/doxygen/InstrTypes_8h_source.html#l01419), we can directly get call operands without even need to extract the call instruction

- setup LLVM
```bash
git clone https://github.com/llvm/llvm-project.git
cd llvm-project
mkdir build && cd build
cmake -G "Unix Makefiles" -DCMAKE_BUILD_TYPE="Release" -DLLVM_ENABLE_PROJECTS="clang;clang-tools-extra;compiler-rt;lldb;lld" DLLVM_ENABLE_RUNTIMES="libcxx;libcxxabi" ../llvm
make -j 4
```

- build passes
link from utaustin [here](https://www.cs.utexas.edu/~pingali/CS380C/2020/assignments/llvm-guide.html)
link from original llvm website [here](https://llvm.org/docs/WritingAnLLVMPass.html)
```bash
cp path/to/dmon/llvm-passes/selective-prefetch ../llvm/lib/Transforms
vim ../llvm/lib/Transforms/CMakeLists.txt 
```
at the end of the file, add 
```
add_subdirectory(selective-prefetch)
```

- how to identify the `faas_func_call` function
	+ some docs that might be useful in the future
	+ [source level debugging](https://llvm.org/docs/SourceLevelDebugging.html): LLVM debug information always provides information to accurately read the source-level state of the program.
	+ [llvm.dbg.value](https://llvm.org/docs/SourceLevelDebugging.html#llvm-dbg-value)	
	+ [debug info assignment tracking](https://llvm.org/docs/AssignmentTracking.html): This intrinsic marks the position in IR where a source assignment occurred. It encodes the value of the variable.
	+ why do we identify serverless function invocation in this way?
		* because the format of function call invocation is fixed. 
		* see [the nightcore code](https://github.com/ut-osa/nightcore/blob/asplos-release/include/faas/worker_v1_interface.h#L22)

```c++
typedef int (*faas_invoke_func_fn_t)(
    void* caller_context, const char* func_name,
    const char* input_data, size_t input_length,
    const char** output_data, size_t* output_length);
```


### faster inter-thread synchronization


### misc
- [vtable](https://llvm.org/devmtg/2021-11/slides/2021-RelativeVTablesinC.pdf) again 

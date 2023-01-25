---
layout: post
title:  "2023-1-21 How to build llvm pass"
date:   2023-01-20 10:53:46 -0500
categories: data-cache 
---
### setup LLVM
```bash
git clone https://github.com/llvm/llvm-project.git
cd llvm-project
mkdir build && cd build
cmake -G "Unix Makefiles" -DCMAKE_BUILD_TYPE="Release" -DLLVM_ENABLE_PROJECTS="clang;clang-tools-extra;compiler-rt;lldb;lld" DLLVM_ENABLE_RUNTIMES="libcxx;libcxxabi" ../llvm
make -j 4
```

### build passes
```bash
cp path/to/dmon/llvm-passes/selective-prefetch ../llvm/lib/Transforms
vim ../llvm/lib/Transforms/CMakeLists.txt 
```
at the end of the file, add 
```
add_subdirectory(selective-prefetch)
```

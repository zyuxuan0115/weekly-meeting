---
layout: post
title:  "2024-01-07 outline + Merge Func"
date:   2024-01-07 1:53:46 -0500
categories: serverless functions
---

### Insert new member to a class
- see [LibTooling and LibASTMatchers tutorial](https://stackoverflow.com/questions/36439248/insert-int-variable-into-the-class-using-llvm-pass-or-clang)

### things about CMake
- [set](https://cmake.org/cmake/help/latest/command/set.html#set-cache-entry)
- [add_library](https://cmake.org/cmake/help/latest/command/add_library.html)
- [target_include_directory](https://cmake.org/cmake/help/latest/command/target_include_directories.html)
- [CMAKE_CURRENT_SOURCE_DIR](https://cmake.org/cmake/help/latest/variable/CMAKE_CURRENT_SOURCE_DIR.html)

### ways to link static library in C
- [stackoverflow](https://stackoverflow.com/questions/1705961/how-to-link-to-a-static-library-in-c)
- [difference between .so and .a](https://unix.stackexchange.com/questions/13192/what-is-the-difference-between-a-and-so-file)
- [tricks when linking boost library](https://stackoverflow.com/questions/23137637/linker-error-while-linking-boost-log-tutorial-undefined-references)

### clang-rename
- [tutorial](https://clang.llvm.org/extra/clang-rename.html)

### papers
- [Serverless in the wild](https://dl.acm.org/doi/abs/10.5555/3489146.3489160)
- [Peeking Behind the curtains of Serverless platform](https://www.usenix.org/conference/atc18/presentation/wang-liang)
- [nightcore](https://www.cs.utexas.edu/users/witchel/pubs/jia21asplos-nightcore.pdf)

### overhead of generating a new libUniqueIdService.so

```
real	1m15.244s
user	1m9.665s
sys	0m2.799s
```

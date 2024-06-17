---
layout: post
title:  "2024-06-13 distributed tracing"
date:   2024-06-13 1:53:49 -0500
categories: serverless
---
### Deleting useless functions from IR?
#### LLVM's strip-dead-prototypes
- [strip-dead-prototypes's link](https://www.llvm.org/docs/Passes.html#strip-dead-prototypes-strip-unused-function-prototypes)
- This pass loops over all of the functions in the input module, looking for dead declarations and removes them. Dead declarations are declarations of functions for which no implementation is available (i.e., declarations for unused library functions).
	+ code is [here](https://llvm.org/doxygen/StripDeadPrototypes_8cpp_source.html)
  + to enable this pass, only need to add `-passes=strip-dead-prototypes` 

#### Implib.so
- [Implib.so's github repo](https://github.com/yugr/Implib.so/tree/master)
  + On Linux, if you link against shared library you normally use `-lxyz` compiler option which makes your application depend on `libxyz.so`. This would cause `libxyz.so` to be forcedly loaded at program startup (and its constructors to be executed) even if you never call any of its functions.
  + What this `Implib.so` do is
    * provides all necessary symbols to make linker happy
    * loads wrapped library on first call to any of its functions
    * redirects calls to library symbols

### Distributed tracing on OpenFaaS
[openfaas-tracing-walkthrough](https://github.com/LucasRoesler/openfaas-tracing-walkthrough)

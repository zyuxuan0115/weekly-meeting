---
layout: post
title:  "2024-06-13 distributed tracing"
date:   2024-06-13 1:53:49 -0500
categories: serverless
---
### Deleting useless functions from IR?
- [strip-dead-prototypes](https://www.llvm.org/docs/Passes.html#strip-dead-prototypes-strip-unused-function-prototypes)
  + This pass loops over all of the functions in the input module, looking for dead declarations and removes them. Dead declarations are declarations of functions for which no implementation is available (i.e., declarations for unused library functions).
  + code is [here](https://llvm.org/doxygen/StripDeadPrototypes_8cpp_source.html)
  + to enable this pass, only need to add `-passes=strip-dead-prototypes` 


### Distributed tracing on OpenFaaS
[openfaas-tracing-walkthrough](https://github.com/LucasRoesler/openfaas-tracing-walkthrough)

---
layout: post
title:  "2023-3-27 pg^2+perf stat"
date:   2023-3-27 7:53:46 -0500
categories: data-cache 
---

### the perf results of pr

![IPC2](/assets/2023-03-27/pr-pref-dist-exe.png) 

![IPC3](/assets/2023-03-27/pr-pref-dist-IPC.png)

![IPC1](/assets/2023-03-27/pr-IPC-exe.png) 

### make pg^2 work for pr
- scan through all prefetch distances from 5 to 100 with a stride of 5 at runtime
- then pick the prefetch distance that has the highest IPC
- the overall time for scanning and prefetch injestion is 2.4 seconds.
- the execution time is the same as the optimal solution
 
### DMon related
- successfully get DMon registered to opt.
	+ can print the error msg we embeded to the pass
	+ but still doesn't run the optimization (insert prefetch)
- The thing I detected is that
	+ LoopDataPrefetch doesn't insert prefetch either. 
	+ I added 2 lines to print the results
```cpp
  if (skipFunction(F)){
    llvm::errs()<<"@@@ this function is skipped\n";
    llvm::errs()<<F<<"\n";
    return false;
  }
```
	+ the `if (skipFunction(F))` in this [line](https://github.com/upenn-acg/floar/blob/master/dmon/llvm-passes/LoopDataPrefetch.cpp#L201) skips the function that has loops.
- I comment out the `skipFunction` check.
- Another thing is 	
	+ LoopDataPrefetch cannot pass the following check either 
```cpp
  const SCEV *LSCEV = SE->getSCEV(PtrValue);
  const SCEVAddRecExpr *LSCEVAddRec = dyn_cast<SCEVAddRecExpr>(LSCEV);
  if (!LSCEVAddRec)
    continue;
```
	+ it means that none of the instructions in the for loop of `stride_benchmark.cc` can pass this check.

- so I need to check why this `skipFunction()` function skip that main function.
	+ the header that declares this function is in `llvm/include/llvm/Pass.h`
	+ the body of the function is in `llvm/lib/IR/Pass.cpp`

### Soyoon's next steps

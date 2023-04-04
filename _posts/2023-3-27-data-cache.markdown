---
layout: post
title:  "2023-3-27 pg^2+perf stat"
date:   2023-3-27 7:53:46 -0500
categories: data-cache 
---
### How pg^2 works
![d1.png](/assets/2023-03-27/d1.png)

### The perf results of pr (perf stat time = 500 ms)

![IPC2](/assets/2023-03-27/pr-pref-dist-exe.png) 

![IPC3](/assets/2023-03-27/pr-pref-dist-IPC.png)

![IPC1](/assets/2023-03-27/pr-IPC-exe.png) 

### Make pg^2 work for pagerank
- scan through all prefetch distances from 5 to 100 with a stride of 5 at runtime
- then pick the prefetch distance that has the highest IPC
- the execution time of `pg2-pagerank` is the same as the optimal solution
- the overall time for scanning and prefetch injestion is 0.62 seconds.
	+ time spends on different parts
		* perf stat time 0.01 × 20 = 0.2 seconds
		* ptrace system call time 0.0035 × 20 = 0.068 seconds
	+ is this correct?
		* No. Although `perf stat` attached to `pagerank` for just 0.01 second (10 ms)
		* the time spends on `perf stat` is <strong>25</strong> ms
		* ![s2](/assets/2023-03-27/screenshot2.png)

| | perf stat |	ptrace | insert prefetch | change <br>prefecth dist| overall<br>(20 rounds) |
|:---:|:---:|:---:|:---:|:---:|:---:|
| time (ms) | 25 | 3.5| 1.3| 1.2|	620 |

![time1](/assets/2023-03-27/Time1.png)


### A shorter perf stat time? 
- what if `perf stat` time is 0.1 ms?
	+ `perf stat ... -- sleep 0.0001`
	+ ![s1](/assets/2023-03-27/Screenshot1.png)
	+ compared with `perf stat ... -- sleep 0.01`
	+ ![s2](/assets/2023-03-27/screenshot2.png)

| | perf stat |	ptrace | insert prefetch | change <br>prefecth dist| overall<br>(20 rounds) |
|:---:|:---:|:---:|:---:|:---:|:---:|
| time (ms) | 15 | 3.5| 1.3| 1.2|	400 | 

![time2](/assets/2023-03-27/Time2.png)

![ipc-exe-2](/assets/2023-03-27/pr-IPC-exe-0.1ms.png)

![0.01](/assets/2023-03-27/0.01.png)

![0.02](/assets/2023-03-27/0.02.png)

![0.05](/assets/2023-03-27/0.05.png)

![0.1](/assets/2023-03-27/0.1.png)

![0.2](/assets/2023-03-27/0.2.png)

![0.3](/assets/2023-03-27/0.3.png)

![0.5](/assets/2023-03-27/0.5.png)

- what if `perf stat` time is 5 ms?

| | perf stat |	ptrace | insert prefetch | change <br>prefecth dist| overall<br>(20 rounds) |
|:---:|:---:|:---:|:---:|:---:|:---:|
| time (ms) | 19 | 3.5| 1.3| 1.2|	400 | 

![time3](/assets/2023-03-27/Time3.png)

![ipc-exe-3](/assets/2023-03-27/pr-IPC-exe-5ms.png)

### How to decide when to stop profiling
![v200000d200](/assets/2023-01-24/v200000-d200.png)

![normailzed](/assets/2023-01-24/normalized.png)

 
### DMon related
- successfully get DMon registered to opt.
	+ can print the error msg we embeded to the pass
	+ but still doesn't run the optimization (insert prefetch)
	+ replaced the orignal `LoopDataPrefetch` code with DMon's code and see what kind of instructions can pass the check. 
- The thing I found 
	+ `LoopDataPrefetch` doesn't insert prefetch either. 
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
	+ LoopDataPrefetch cannot pass the [following check](https://github.com/upenn-acg/floar/blob/master/dmon/llvm-passes/LoopDataPrefetch.cpp#L308) either 
```cpp
  const SCEV *LSCEV = SE->getSCEV(PtrValue);
  const SCEVAddRecExpr *LSCEVAddRec = dyn_cast<SCEVAddRecExpr>(LSCEV);
  if (!LSCEVAddRec)
    continue;
```
	+ it means that none of the instructions in the for loop of `stride_benchmark.cc` can pass this check.
- I checked the output of `const SCEV *LSCEV = SE->getSCEV(PtrValue);`
	+ the only instruction that has `*LSCEV` is `%4 = load i8, i8* %arrayidx, align 1, !dbg !964`
```
%idxprom = sext i32 %3 to i64, !dbg !964
%arrayidx = getelementptr inbounds i8, i8* %2, i64 %idxprom, !dbg !964
%4 = load i8, i8* %arrayidx, align 1, !dbg !964
@@@ PtrValue:   %arrayidx = getelementptr inbounds i8, i8* %2, i64 %idxprom, !dbg !964
@@@ LSCEV: ((sext i32 %3 to i64) + %2)<nsw>
```
	+ I checked why this `((sext i32 %3 to i64) + %2)<nsw>` cannot be `dyn_cast`ed to `SCEVAddRecExpr`
	+ to perform `<dyn_cast>`, the `*LSCEV` whose type is base class `SCEV`, should have the type `scAddRecExpr`. 
		* see this [file](https://llvm.org/doxygen/ScalarEvolutionExpressions_8h_source.html)'s line 419 
		* the enum of `SCEV`'s type are listed [here](https://llvm.org/doxygen/namespacellvm.html#ad409632178dd00316abd4c35d3e14b5e) 		
	+ I check the type of this `((sext i32 %3 to i64) + %2)<nsw>` by `llvm::errs()<<LSCEV->getSCEVType()<<"\n";` 
		* and find that its type is `scSignExtend`
		* so it cannot be casted to `SCEVAddRecExpr` 
- other resources about `SCEV`
	* this [link](https://llvm.org/devmtg/2018-04/slides/Absar-ScalarEvolution.pdf)


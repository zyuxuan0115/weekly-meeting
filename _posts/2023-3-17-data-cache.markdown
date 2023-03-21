---
layout: post
title:  "2023-3-17 pg^2 performance counter"
date:   2023-3-17 7:53:46 -0500
categories: data-cache 
---

### the perf results of pr
![LLC](/assets/2023-03-17/LLC.png) 

![IPC](/assets/2023-03-17/IPC.png) 

![sw-prefetch-PKI](/assets/2023-03-17/sw_prefetch_access.t0_PKI.png)

![l2-rqsts.pf-PKI](/assets/2023-03-17/l2_rqsts.pf_PKI.png)

![l2-rqsts.pf-hit-pct](/assets/2023-03-17/l2_rqsts.pf_hit_percentage.png)

### DMon related
- successfully get DMon registered to opt.
	+ can print the error msg we embeded to the pass
	+ but still doesn't run the optimization (insert prefetch)
- The thing I detected is that
	+ the `if (skipFunction(F))` in this [line](https://github.com/upenn-acg/floar/blob/master/dmon/llvm-passes/LoopDataPrefetch.cpp#L201) skips the function that has loops.
	+ I added 2 lines to print the results
```cpp
  if (skipFunction(F)){
    llvm::errs()<<"@@@ this function is skipped\n";
    llvm::errs()<<F<<"\n";
    return false;
  }
```
	+ the output is

```
@@@ this function is skipped

; Function Attrs: noinline norecurse optnone uwtable
define dso_local i32 @main(i32 %argc, i8** %argv) #4 {
entry:
  %retval = alloca i32, align 4
  %argc.addr = alloca i32, align 4
  %argv.addr = alloca i8**, align 8
  %kLineSize = alloca i32, align 4
  %kStride = alloca i32, align 4
  %kArraySize = alloca i64, align 8
  %kIterations = alloca i32, align 4
  %buffer = alloca i8*, align 8
  %position = alloca i32, align 4
  %loop = alloca i32, align 4
  store i32 0, i32* %retval, align 4
  store i32 %argc, i32* %argc.addr, align 4
  store i8** %argv, i8*** %argv.addr, align 8
  store i32 64, i32* %kLineSize, align 4
  store i32 7, i32* %kStride, align 4
  store i64 16777216, i64* %kArraySize, align 8
  store i32 2000000000, i32* %kIterations, align 4
  %call = call i8* @_Znam(i64 16777216) #7
  store i8* %call, i8** %buffer, align 8
  %0 = load i8*, i8** %buffer, align 8
  call void @llvm.memset.p0i8.i64(i8* align 1 %0, i8 7, i64 16777216, i1 false)
  call void asm sideeffect "", "~{memory},~{dirflag},~{fpsr},~{flags}"() #3, !srcloc !2
  store i32 0, i32* %position, align 4
  store i32 0, i32* %loop, align 4
  br label %for.cond

for.cond:                                         ; preds = %for.inc, %entry
  %1 = load i32, i32* %loop, align 4
  %cmp = icmp slt i32 %1, 2000000000
  br i1 %cmp, label %for.body, label %for.end

for.body:                                         ; preds = %for.cond
  %2 = load i8*, i8** %buffer, align 8
  %3 = load i32, i32* %position, align 4
  %idxprom = sext i32 %3 to i64
  %arrayidx = getelementptr inbounds i8, i8* %2, i64 %idxprom
  %4 = load i8, i8* %arrayidx, align 1
  %conv = zext i8 %4 to i32
  %mul = mul nsw i32 %conv, 64
  %5 = load i32, i32* %position, align 4
  %add = add nsw i32 %5, %mul
  store i32 %add, i32* %position, align 4
  %6 = load i32, i32* %position, align 4
  %conv1 = sext i32 %6 to i64
  %and = and i64 %conv1, 16777215
  %conv2 = trunc i64 %and to i32
  store i32 %conv2, i32* %position, align 4
  br label %for.inc

for.inc:                                          ; preds = %for.body
  %7 = load i32, i32* %loop, align 4
  %inc = add nsw i32 %7, 1
  store i32 %inc, i32* %loop, align 4
  br label %for.cond

for.end:                                          ; preds = %for.cond
  call void asm sideeffect "", "~{memory},~{dirflag},~{fpsr},~{flags}"() #3, !srcloc !3
  %call3 = call dereferenceable(272) %"class.std::basic_ostream"* @_ZStlsISt11char_traitsIcEERSt13basic_ostreamIcT_ES5_PKc(%"class.std::basic_ostream"* dereferenceable(272) @_ZSt4cerr, i8* getelementptr inbounds ([9 x i8], [9 x i8]* @.str, i32 0, i32 0))
  %8 = load i32, i32* %position, align 4
  %call4 = call dereferenceable(272) %"class.std::basic_ostream"* @_ZNSolsEi(%"class.std::basic_ostream"* %call3, i32 %8)
  %call5 = call dereferenceable(272) %"class.std::basic_ostream"* @_ZNSolsEPFRSoS_E(%"class.std::basic_ostream"* %call4, %"class.std::basic_ostream"* (%"class.std::basic_ostream"*)* @_ZSt4endlIcSt11char_traitsIcEERSt13basic_ostreamIT_T0_ES6_)
  ret i32 0
}
```

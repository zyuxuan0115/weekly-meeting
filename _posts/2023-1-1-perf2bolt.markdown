---
layout: post
title:  "2022-1-1 extracting reversed BAT"
date:   2023-1-1 10:53:46 -0500
categories: cont-opt 
---

### Extract reversed BAT from a BOLTed binary
- The next step is to get the reversed BAT from the BOLTed binary, when we run  
`perf2bolt --ignore-build-id --cont-opt -p perf2.data -o perf2.fdata mysqld.bolt`
    + where `perf2.data` is collected from <strong>C1 round</strong> from the mysqld which has text section changed by OCOLOS, 
    + and `mysqld.bolt` is produced in the <strong>C0 round</strong>
- I noticed that in [BAT-dump](https://github.com/zyuxuan0115/llvm-project/blob/main/bolt/tools/bat-dump/bat-dump.cpp)'s code 
    + when it dumps the BAT table, it calls [dumpBATFor()](https://github.com/zyuxuan0115/llvm-project/blob/main/bolt/tools/bat-dump/bat-dump.cpp#L81)
    + and this `dumpBATfor()` calls [BAT.parse()](https://github.com/zyuxuan0115/llvm-project/blob/main/bolt/lib/Profile/BoltAddressTranslation.cpp#L240) to update the empty `Maps`.
- We can add a [BAT.parseReverse()]() to update the empty [ReversedMap]().
    + when llvm-bolt produce the optimal binary, this `ReversedMap` is also used to store the BAT table in `RewriteInstance.cpp`'s `RewriteInstance::encodeReversedBATSection()` that we added before. This is because
        * `encodeReversedBATSection()` calls `BAT->writeReverse()`
        * and `BAT->writeReverse()` updates this `ReversedMap`
- The next question is how to run this `BAT->parseReverse()`.
    + [RewriteInstance::readSpecialSections()](https://github.com/zyuxuan0115/llvm-project/blob/main/bolt/lib/Rewrite/RewriteInstance.cpp#L1593) uses [BAT->parse()](https://github.com/zyuxuan0115/llvm-project/blob/main/bolt/lib/Rewrite/RewriteInstance.cpp#L1642). 
    + [dumpBATFor()](https://github.com/zyuxuan0115/llvm-project/blob/main/bolt/tools/bat-dump/bat-dump.cpp#L81) also uses [BAT.parse()](https://github.com/zyuxuan0115/llvm-project/blob/main/bolt/tools/bat-dump/bat-dump.cpp#L112)
        * These 2 examples can be considered when we use `BAT->parseReverse()`, in `perf2bolt`'s [DataAggregator](https://github.com/zyuxuan0115/llvm-project/blob/main/bolt/lib/Profile/DataAggregator.cpp)
        * Because in `DataAggregator`, `BinaryContext BC` already exists, I choose to use [RewriteInstance::readSpecialSections()](https://github.com/zyuxuan0115/llvm-project/blob/main/bolt/lib/Rewrite/RewriteInstance.cpp#L1593)'s way to run `BAT->parseReverse()`.
    + add [readReversedBATSections()](https://github.com/zyuxuan0115/llvm-project/blob/main/bolt/lib/Profile/DataAggregator.cpp#L665) to `DataAggregator`
        * This `readReversedBATSections()` will get the reversed BAT from BOLTed binary's note section.
        * And then will store the reversed BAT into [ReversedMap](https://github.com/zyuxuan0115/llvm-project/blob/main/bolt/include/bolt/Profile/BoltAddressTranslation.h#L132)

### With <strong>reversed BAT</strong>, modify LBR profile collected from <strong>C1</strong> round
```
2833111           46021c0 
0x206bd10/0x4601f44/P/-/-/1  
0x206bc3b/0x206bd10/P/-/-/90  
0x37a904c/0x206bc1d/P/-/-/8  
0x37a9064/0x37a904a/P/-/-/3  
...
```

- On 12-12-2022's post, we know that in [parseBranchEvents()](https://github.com/zyuxuan0115/llvm-project/blob/main/bolt/lib/Profile/DataAggregator.cpp#L1425), it calls [parseBranchSample()](https://github.com/zyuxuan0115/llvm-project/blob/main/bolt/lib/Profile/DataAggregator.cpp#L1095)
    + `parseBranchSample()` will first extract PID from the `perf script`'s output
        * compare this PID with the PID stored in `BinaryMMapInfo`
    + `parseBranchSample()` will then extract IP (instruction) pointer from the `perf script`'s output       
    + After getting the `PID` and `Address` of the branch, `parseBranchSample()` calls [parseLBREntry()](https://github.com/zyuxuan0115/llvm-project/blob/main/bolt/lib/Profile/DataAggregator.cpp#L1012)
    + In `parseLBREntry()`, it parses the string by `/`, and then records:
        * the first address to `Res.From`
        * the second address to `Res.To`
        * the third element into `Res.Mispred`
- How the data is recorded
    + In `parseLBREntry()` 
        * the addresses from LBR sample will be recorded into [LBREntry](https://github.com/zyuxuan0115/llvm-project/blob/main/bolt/include/bolt/Profile/DataReader.h#L34)
        * then the `LBREntry` as the return value will be passed to `parseBranchSample()`
    + In `parseBranchSample()`
        * the `LBREntry` will be pushed to [PerfBranchSample](https://github.com/zyuxuan0115/llvm-project/blob/main/bolt/include/bolt/Profile/DataAggregator.h#L83)
        * then the `PerfBranchSample` as the return value will be passed to `parseBranchEvents()`
    + In `parseBranchEvents()`, it will discard some of the abnormal/illegal samples (such as our case where profile collected from C1 round)
        * it's good to add
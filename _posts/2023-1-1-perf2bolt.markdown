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
    + These 2 examples can be considered when we use BAT->parseReverse(), in `perf2bolt`'s [DataAggregator](https://github.com/zyuxuan0115/llvm-project/blob/main/bolt/lib/Profile/DataAggregator.cpp)
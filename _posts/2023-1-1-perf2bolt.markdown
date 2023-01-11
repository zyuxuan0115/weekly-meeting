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
    + `parseBranchSample()` will then extract IP (instruction pointer） from the `perf script`'s output       
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
        * it's good to add `DataAggregator::readReversedBATSections()` at the beginning of this `parseBranchEvents()`, in order to get the reversed BAT from Binary Context.
        * then in `parseLBREntry()`'s updating of `LBREntry`, we are able to change the original address into the BOLTed address according to reversed BAT.
- The code added to change the LBR record
    + 
    + store the reversed BAT into a hash map in order to perform lookup easily
    + in [parseLBREntry()](), change the address if the address of the instruction occurs in the reversed BAT

### Problem needs to be fixed
After switching to our modified version of `llvm-bolt`, I got the following <strong>mmap()</strong> error when OCOLOS inserted code to mysqld's text section:
```
[tracee (lib)] target addr = 0x62b1700, len=13
[tracee (lib)] mmap failed
[tracee (lib)] error: File exists
```

To check why mmap() has such an error, I first used `gdb` to disassemble the BOLTed function at `0x62b1700`, and found there is a function at the address.
```gdb
(gdb) disas 0x62b1700
Dump of assembler code for function _fini:
   0x00000000062b1700 <+0>:	endbr64
   0x00000000062b1704 <+4>:	sub    $0x8,%rsp
   0x00000000062b1708 <+8>:	add    $0x8,%rsp
   0x00000000062b170c <+12>:	retq
End of assembler dump.
``` 

- Does BOLT (my modified version) really place the BOLTed functions at the address that is higher than 0x60000000?
    + Yes, it does.
    + This is the result from `nm -S mysqld.bolt`
```
0000000004b2e080 T _Z17is_sqlstate_validPK16MYSQL_LEX_STRING
0000000004c8b0c0 T _Z17is_updatable_viewP3THDP10TABLE_LIST
0000000004728e1a T _Z17is_valid_log_namePKcm
0000000005a7e340 T _Z17lf_dynarray_valueP11LF_DYNARRAYj
0000000005a7df48 T _Z17lf_pinbox_destroyP9LF_PINBOX
00000000061e7e8a T _Z17localtime_to_TIMEP10MYSQL_TIMEPK2tm
0000000005e11184 T _Z17lock_get_mode_strPK9ib_lock_t
0000000005e110c4 t _Z17lock_get_mode_strPK9ib_lock_t.cold
0000000005e03e88 T _Z17lock_get_table_idPK9ib_lock_t
0000000005e03e56 T _Z17lock_get_type_strPK9ib_lock_t
0000000004bc8a50 T _Z17lock_plugin_mutexv
0000000005e03840 T _Z17lock_rec_get_prevPK9ib_lock_tm
0000000005e0369c T _Z17lock_rec_trx_waitP9ib_lock_tmm
0000000005e30b80 T _Z17log_buffer_resizeR5log_tm
000000000618c280 T _Z17log_builtins_exitv
000000000618d61e T _Z17log_builtins_initv
000000000618d600 t _Z17log_builtins_initv.cold
```

- The address space of the <strong>original binary</strong> which is waiting for `OCOLOS`'s code replacement
```
  1 00200000-01b22000 r--p 00000000 09:00 126354168        /usr/local/mysql/bin/mysqld
  2 01b22000-038cb000 r-xp 01921000 09:00 126354168        /usr/local/mysql/bin/mysqld
  3 038cb000-03a40000 r--p 036c9000 09:00 126354168        /usr/local/mysql/bin/mysqld
  4 03a40000-03dcb000 rw-p 0383d000 09:00 126354168        /usr/local/mysql/bin/mysqld
  5 03dcb000-042fd000 rw-p 00000000 00:00 0
  6 0471b000-07043000 rw-p 00000000 00:00 0                [heap]
```
- The address space of the <strong>BOLTed binary</strong> which `has reversed BAT` & `BAT` in its address space. 
```
  1 00200000-01b22000 r--p 00000000 09:00 126364470        /usr/local/mysql/bin/mysqld.bolt
  2 01b22000-038cb000 r-xp 01921000 09:00 126364470        /usr/local/mysql/bin/mysqld.bolt
  3 038cb000-03a40000 r--p 036c9000 09:00 126364470        /usr/local/mysql/bin/mysqld.bolt
  4 03a40000-03dcb000 rw-p 0383d000 09:00 126364470        /usr/local/mysql/bin/mysqld.bolt
  5 03dcb000-042fd000 rw-p 00000000 00:00 0
  6 04400000-069db000 r-xp 04200000 09:00 126364470        /usr/local/mysql/bin/mysqld.bolt
  7 071a0000-09ac8000 rw-p 00000000 00:00 0                [heap]
```
- The address space of the <strong>BOLTed binary</strong> which `has NO reversed BAT` but `has BAT` in its address space.
```
  1 00200000-01b22000 r--p 00000000 09:00 126364470       /usr/local/mysql/bin/mysqld.bolt
  2 01b22000-038cb000 r-xp 01921000 09:00 126364470       /usr/local/mysql/bin/mysqld.bolt
  3 038cb000-03a40000 r--p 036c9000 09:00 126364470       /usr/local/mysql/bin/mysqld.bolt
  4 03a40000-03dcb000 rw-p 0383d000 09:00 126364470       /usr/local/mysql/bin/mysqld.bolt
  5 03dcb000-042fd000 rw-p 00000000 00:00 0
  6 04400000-04b1b000 r-xp 04200000 09:00 126364470       /usr/local/mysql/bin/mysqld.bolt
  7 0649e000-08dc6000 rw-p 00000000 00:00 0               [heap]
```
- The address space of the <strong>BOLTed binary</strong> which has neither `reversed BAT` nor `BAT` in its address space
```
  1 00200000-01b22000 r--p 00000000 09:00 126364470       /usr/local/mysql/bin/mysqld.bolt
  2 01b22000-038cb000 r-xp 01921000 09:00 126364470       /usr/local/mysql/bin/mysqld.bolt
  3 038cb000-03a40000 r--p 036c9000 09:00 126364470       /usr/local/mysql/bin/mysqld.bolt
  4 03a40000-03dcb000 rw-p 0383d000 09:00 126364470       /usr/local/mysql/bin/mysqld.bolt
  5 03dcb000-042fd000 rw-p 00000000 00:00 0
  6 04400000-04b1b000 r-xp 04200000 09:00 126364470       /usr/local/mysql/bin/mysqld.bolt
  7 05ede000-08806000 rw-p 00000000 00:00 0               [heap]
```


    

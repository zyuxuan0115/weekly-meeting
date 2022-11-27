---
layout: post
title:  "2022-11-28 perf.data for BOLTed binary"
date:   2022-11-24 12:53:46 -0500
categories: cont-opt
---

- the output of `perf script --show-mmap-events`
```
     mysqld.bolt     0     0.000000: PERF_RECORD_MMAP2 1692770/1692770: [0xcdb000(0x1da5000) @ 0x8db000 09:00 179639298 0]: r-xp /usr/local/mysql/bin/mysqld.bolt
     mysqld.bolt     0     0.000000: PERF_RECORD_MMAP2 1692770/1692770: [0x4600000(0x25b2000) @ 0x4200000 09:00 179639298 0]: r-xp /usr/local/mysql/bin/mysqld.bolt
     mysqld.bolt     0     0.000000: PERF_RECORD_MMAP2 1692770/1692770: [0x7f054bb10000(0x6000) @ 0x2000 09:00 178783333 0]: r-xp /usr/lib/x86_64-linux-gnu/libnss_sss.so.2
     mysqld.bolt     0     0.000000: PERF_RECORD_MMAP2 1692770/1692770: [0x7f0557c31000(0x4f000) @ 0x1c000 09:00 178782954 0]: r-xp /usr/lib/x86_64-linux-gnu/libssl.so.1.1
     mysqld.bolt     0     0.000000: PERF_RECORD_MMAP2 1692770/1692770: [0x7f0557caa000(0x4000) @ 0x2000 09:00 178790204 0]: r-xp /usr/lib/x86_64-linux-gnu/librt-2.31.so
     mysqld.bolt     0     0.000000: PERF_RECORD_MMAP2 1692770/1692770: [0x7f0557cc9000(0x4000) @ 0x2000 09:00 179636181 0]: r-xp /usr/local/mysql/lib/plugin/component_reference_cache.so
     mysqld.bolt     0     0.000000: PERF_RECORD_MMAP2 1692770/1692770: [0x7f0557cd4000(0x23000) @ 0x1000 09:00 178790185 0]: r-xp /usr/lib/x86_64-linux-gnu/ld-2.31.so
     mysqld.bolt     0     0.000000: PERF_RECORD_MMAP2 1692770/1692770: [0x7ffda13bf000(0x1000) @ 0 00:00 0 0]: r-xp [vdso]
     mysqld.bolt     0     0.000000: PERF_RECORD_MMAP2 1692770/1692770: [0xffffffffff600000(0x1000) @ 0 00:00 0 0]: --xp [vsyscall] 
```
```
      connection 1692829 31816801.787443:          1 cycles:u:      7f055788636c __libc_read+0x4c (/usr/lib/x86_64-linux-gnu/libpthread-2.31.so)
      connection 1692829 31816801.787452:          1 cycles:u:      7f055788636c __libc_read+0x4c (/usr/lib/x86_64-linux-gnu/libpthread-2.31.so)
      connection 1692829 31816801.787458:          4 cycles:u:      7f055788636c __libc_read+0x4c (/usr/lib/x86_64-linux-gnu/libpthread-2.31.so)
      connection 1692829 31816801.787463:         26 cycles:u:      7f055788636c __libc_read+0x4c (/usr/lib/x86_64-linux-gnu/libpthread-2.31.so)
      connection 1692829 31816801.787468:        179 cycles:u:      7f055788636c __libc_read+0x4c (/usr/lib/x86_64-linux-gnu/libpthread-2.31.so)
      connection 1692829 31816801.787474:       1190 cycles:u:      7f0557886374 __libc_read+0x54 (/usr/lib/x86_64-linux-gnu/libpthread-2.31.so)
      connection 1692829 31816801.787481:       7668 cycles:u:      7f05579e79d9 BIO_clear_flags+0x9 (/usr/lib/x86_64-linux-gnu/libcrypto.so.1.1)
      connection 1692829 31816801.787490:      43105 cycles:u:           4802794 pfs_end_idle_wait_v1+0x14 (/usr/local/mysql/bin/mysqld.bolt)
      connection 1692829 31816801.787510:     175803 cycles:u:           485e2d7 CreateIteratorFromAccessPath+0x3cd (/usr/local/mysql/bin/mysqld.bolt)
      connection 1692829 31816801.787639:     431505 cycles:u:           4871596 btr_cur_search_to_nth_level+0x3794 (/usr/local/mysql/bin/mysqld.bolt)
      connection 1692829 31816801.787968:     482561 cycles:u:           4810d3a [unknown] (/usr/local/mysql/bin/mysqld.bolt)
```

- the output of `perf script --show-task-events`
```
      connection 1692829 31816801.857336:     359251 cycles:u:      7f0557c39b94 [unknown] (/usr/lib/x86_64-linux-gnu/libssl.so.1.1)
      connection 1692830 31816801.857465:     348516 cycles:u:      7f0557c4de17 SSL_write+0x27 (/usr/lib/x86_64-linux-gnu/libssl.so.1.1)
      connection 1692828 31816801.857528:     427007 cycles:u:      7f0557b0cd7c [unknown] (/usr/lib/x86_64-linux-gnu/libcrypto.so.1.1)
      connection 1692829 31816801.857584:     359190 cycles:u:           481d79f validate_use_secondary_engine+0x2df (/usr/local/mysql/bin/mysqld.bolt)
      connection 1692827 31816801.857589:     365559 cycles:u:           484e56c pfs_end_table_io_wait_v1+0xd8 (/usr/local/mysql/bin/mysqld.bolt)
      connection 1692830 31816801.857728:     352942 cycles:u:           484f202 calc_length_and_keyparts+0x82 (/usr/local/mysql/bin/mysqld.bolt)
      connection 1692828 31816801.857805:     420601 cycles:u:           483f790 JOIN::optimize+0x1050 (/usr/local/mysql/bin/mysqld.bolt)
      connection 1692829 31816801.857837:     359593 cycles:u:           480b7c9 PT_table_factor_table_ident::contextualize+0x7d (/usr/local/mysql/bin/mysqld.bolt)
      connection 1692827 31816801.857842:     363215 cycles:u:           48a7a03 lex_one_token+0xed (/usr/local/mysql/bin/mysqld.bolt)
```

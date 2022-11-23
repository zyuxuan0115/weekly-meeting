---
layout: post
title:  "2022-11-27 perf.data for BOLTed binary"
date:   2022-11-22 12:53:46 -0500
categories: cont-opt
---

	* The output of `perf script -F pid,event,ip`
		```
		```

	* The output of `perf script -F pid,ip,brstack` 
    ```

		```

	* The output of `perf script -F pid,event,addr,ip`
		+ empty
		+ `perf script -F pid,event,addr,ip` on the original binary's perf.data is also empty

	* the output of `perf script --show-mmap-events`
		```
 
		```


	* the output of `perf script --show-task-events`
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

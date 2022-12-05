---
layout: post
title:  "2022-12-05 /proc/PID/maps"
date:   2022-12-01 12:53:46 -0500
categories: cont-opt
---

- `/proc/{mysqld}/maps`

```
  1 00200000-01b23000 r--p 00000000 09:00 179636315                   /usr/local/mysql/bin/mysqld
  2 01b23000-038cd000 r-xp 01922000 09:00 179636315                   /usr/local/mysql/bin/mysqld
  3 038cd000-03a42000 r--p 036cb000 09:00 179636315                   /usr/local/mysql/bin/mysqld
  4 03a42000-03dcd000 rw-p 0383f000 09:00 179636315                   /usr/local/mysql/bin/mysqld
  5 03dcd000-042ff000 rw-p 00000000 00:00 0
  6 0524e000-07b76000 rw-p 00000000 00:00 0                           [heap]
```

- `/proc/{mysqld.bolt pid}/maps`

```
  1 00200000-01b23000 r--p 00000000 09:00 179639298                   /usr/local/mysql/bin/mysqld.bolt
  2 01b23000-038cd000 r-xp 01922000 09:00 179639298                   /usr/local/mysql/bin/mysqld.bolt
  3 038cd000-03a42000 r--p 036cb000 09:00 179639298                   /usr/local/mysql/bin/mysqld.bolt
  4 03a42000-03dcd000 rw-p 0383f000 09:00 179639298                   /usr/local/mysql/bin/mysqld.bolt
  5 03dcd000-042ff000 rw-p 00000000 00:00 0
  6 04400000-04b1b000 r-xp 04200000 09:00 179639298                   /usr/local/mysql/bin/mysqld.bolt
  7 05255000-07b7d000 rw-p 00000000 00:00 0                           [heap] 
```

- `/proc/{mysqld after cope replacement PID}/maps`

```
  1 00200000-01b23000 r--p 00000000 09:00 179636315                   /usr/local/mysql/bin/mysqld
  2 01b23000-01b28000 r-xp 01922000 09:00 179636315                   /usr/local/mysql/bin/mysqld
  3 01b28000-01b2b000 rwxp 01927000 09:00 179636315                   /usr/local/mysql/bin/mysqld
  4 01b2b000-01b40000 r-xp 0192a000 09:00 179636315                   /usr/local/mysql/bin/mysqld
  5 01b40000-01b47000 rwxp 0193f000 09:00 179636315                   /usr/local/mysql/bin/mysqld
  6 01b47000-021da000 r-xp 01946000 09:00 179636315                   /usr/local/mysql/bin/mysqld
  7 021da000-021dc000 rwxp 01fd9000 09:00 179636315                   /usr/local/mysql/bin/mysqld
  8 021dc000-021ea000 r-xp 01fdb000 09:00 179636315                   /usr/local/mysql/bin/mysqld
  9 021ea000-021ec000 rwxp 01fe9000 09:00 179636315                   /usr/local/mysql/bin/mysqld
 10 021ec000-021ed000 r-xp 01feb000 09:00 179636315                   /usr/local/mysql/bin/mysqld
 11 021ed000-021ef000 rwxp 01fec000 09:00 179636315                   /usr/local/mysql/bin/mysqld
 12 021ef000-02bc6000 r-xp 01fee000 09:00 179636315                   /usr/local/mysql/bin/mysqld
 13 02bc6000-02bc9000 rwxp 029c5000 09:00 179636315                   /usr/local/mysql/bin/mysqld
 14 02bc9000-034de000 r-xp 029c8000 09:00 179636315                   /usr/local/mysql/bin/mysqld
 15 034de000-034e0000 rwxp 032dd000 09:00 179636315                   /usr/local/mysql/bin/mysqld
 16 034e0000-03688000 r-xp 032df000 09:00 179636315                   /usr/local/mysql/bin/mysqld
 17 03688000-0368b000 rwxp 03487000 09:00 179636315                   /usr/local/mysql/bin/mysqld
 18 0368b000-03748000 r-xp 0348a000 09:00 179636315                   /usr/local/mysql/bin/mysqld
 19 03748000-0374a000 rwxp 03547000 09:00 179636315                   /usr/local/mysql/bin/mysqld
 20 0374a000-03751000 r-xp 03549000 09:00 179636315                   /usr/local/mysql/bin/mysqld
 21 03751000-03754000 rwxp 03550000 09:00 179636315                   /usr/local/mysql/bin/mysqld
 22 03754000-038cd000 r-xp 03553000 09:00 179636315                   /usr/local/mysql/bin/mysqld
 23 038cd000-038cf000 r--p 036cb000 09:00 179636315                   /usr/local/mysql/bin/mysqld
 24 038cf000-039bb000 rwxp 036cd000 09:00 179636315                   /usr/local/mysql/bin/mysqld
 25 039bb000-039be000 r--p 037b9000 09:00 179636315                   /usr/local/mysql/bin/mysqld
 26 039be000-039e2000 rwxp 037bc000 09:00 179636315                   /usr/local/mysql/bin/mysqld
 27 039e2000-039e3000 r--p 037e0000 09:00 179636315                   /usr/local/mysql/bin/mysqld
 28 039e3000-039f8000 rwxp 037e1000 09:00 179636315                   /usr/local/mysql/bin/mysqld
 29 039f8000-039f9000 r--p 037f6000 09:00 179636315                   /usr/local/mysql/bin/mysqld
 30 039f9000-03a1e000 rwxp 037f7000 09:00 179636315                   /usr/local/mysql/bin/mysqld
 31 03a1e000-03a23000 r--p 0381c000 09:00 179636315                   /usr/local/mysql/bin/mysqld
 32 03a23000-03a42000 rwxp 03821000 09:00 179636315                   /usr/local/mysql/bin/mysqld
 33 03a42000-03dcd000 rw-p 0383f000 09:00 179636315                   /usr/local/mysql/bin/mysqld
 34 03dcd000-042ff000 rw-p 00000000 00:00 0
 35 04600000-04603000 r-xp 00000000 00:00 0
 36 04603000-04607000 r-xp 00000000 00:00 0
 37 04607000-04610000 r-xp 00000000 00:00 0
 38 04610000-04618000 r-xp 00000000 00:00 0
 39 04618000-0461b000 r-xp 00000000 00:00 0
 40 0461b000-0461e000 r-xp 00000000 00:00 0
 41 0461e000-04623000 r-xp 00000000 00:00 0
 42 04623000-04628000 r-xp 00000000 00:00 0
 43 04628000-0462b000 r-xp 00000000 00:00 0
 44 0462b000-04635000 r-xp 00000000 00:00 0
 45 04635000-0463a000 r-xp 00000000 00:00 0
 46 0463a000-0463d000 r-xp 00000000 00:00 0
 47 0463d000-04641000 r-xp 00000000 00:00 0
 48 04641000-04646000 r-xp 00000000 00:00 0
 49 04646000-0464a000 r-xp 00000000 00:00 0
 50 0464a000-0464d000 r-xp 00000000 00:00 0
 51 0464d000-04651000 r-xp 00000000 00:00 0
 52 04651000-04658000 r-xp 00000000 00:00 0
 53 04658000-04660000 r-xp 00000000 00:00 0
 54 04660000-04668000 r-xp 00000000 00:00 0
 55 04668000-0466e000 r-xp 00000000 00:00 0
 56 0466e000-04675000 r-xp 00000000 00:00 0
 57 04675000-046a5000 r-xp 00000000 00:00 0
 58 046a5000-046a9000 r-xp 00000000 00:00 0
 59 046a9000-046af000 r-xp 00000000 00:00 0
 60 046af000-046b9000 r-xp 00000000 00:00 0
 61 046b9000-046c1000 r-xp 00000000 00:00 0
 62 046c1000-046c4000 r-xp 00000000 00:00 0
 63 046c4000-046c7000 r-xp 00000000 00:00 0
 64 046c7000-046cb000 r-xp 00000000 00:00 0
 65 046cb000-046d0000 r-xp 00000000 00:00 0
 66 046d0000-046d4000 r-xp 00000000 00:00 0
 67 046d4000-046dd000 r-xp 00000000 00:00 0
 68 046dd000-046e5000 r-xp 00000000 00:00 0
 69 046e5000-046e9000 r-xp 00000000 00:00 0
 70 046e9000-046f5000 r-xp 00000000 00:00 0
 71 046f5000-046fc000 r-xp 00000000 00:00 0
 72 046fc000-04705000 r-xp 00000000 00:00 0
 73 04705000-04707000 r-xp 00000000 00:00 0
 74 05baa000-084d2000 rw-p 00000000 00:00 0                                  [heap]
```


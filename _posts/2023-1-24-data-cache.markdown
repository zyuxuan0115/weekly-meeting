---
layout: post
title:  "2023-1-24 data cache"
date:   2023-1-24 8:53:46 -0500
categories: data-cache 
---
### STREAM
- According to [STREAM benchmark](https://github.com/jeffhammond/STREAM), we need to set `STREAM_ARRAY_SIZE` to be 4 times the size of the available cache memory
- here is there [example](https://github.com/jeffhammond/STREAM/blob/master/stream.c#L58)
- I run `sudo lshw -C memory` to get the size of L3 cache 
	+ seems like we have 2 L3 cache and each has a size of 35 MB.
```
  *-cache:0
       description: L1 cache
       physical id: 50
       slot: L1 Cache
       size: 1664KiB
       capacity: 1664KiB
       capabilities: synchronous internal write-back instruction
       configuration: level=1
  *-cache:1
       description: L2 cache
       physical id: 51
       slot: L2 Cache
       size: 26MiB
       capacity: 26MiB
       capabilities: synchronous internal varies unified
       configuration: level=2
  *-cache:2
       description: L3 cache
       physical id: 52
       slot: L3 Cache
       size: 35MiB
       capacity: 35MiB
       capabilities: synchronous internal varies unified
       configuration: level=3
  *-cache:3
       description: L1 cache
       physical id: 54
       slot: L1 Cache
       size: 1664KiB
       capacity: 1664KiB
       capabilities: synchronous internal write-back instruction
       configuration: level=1
  *-cache:4
       description: L2 cache
       physical id: 55
       slot: L2 Cache
       size: 26MiB
       capacity: 26MiB
       capabilities: synchronous internal varies unified
       configuration: level=2
  *-cache:5
       description: L3 cache
       physical id: 56
       slot: L3 Cache
       size: 35MiB
       capacity: 35MiB
       capabilities: synchronous internal varies unified
       configuration: level=3
  *-memory UNCLAIMED
       description: Memory controller
       product: C620 Series Chipset Family Power Management Controller
       vendor: Intel Corporation
       physical id: 1f.2
       bus info: pci@0000:00:1f.2
       version: 09
       width: 32 bits
       clock: 33MHz (30.3ns)
       configuration: latency=0
       resources: memory:9d210000-9d213fff
```
- So the final configuration is
	+ 2 x 35MiB x 4 / 8 = 36700160 
	+ the smallest `STREAM_ARRAY_SIZE` is 36,700,160 
     + I set the `STREAM_ARRAY_SIZE` to be `1e10`, and for 1 iteration it runs for 3.5 minutes.
     + there is another parameter [NTIMES](https://github.com/jeffhammond/STREAM/blob/master/stream.c#L108), it can change the number of iterations. I set it to be 40.

### Result when running pagerank with STREAM
![web-stanford](/assets/2023-01-24/web-Stanford.png)
![web-berkstan](/assets/2023-01-24/web-Stanford.png)
![web-NotreDame](/assets/2023-01-24/web-NotreDame.png)



### DMon
- successfully built DMon's LLVM pass by using LLVM 7.0
	+ what is the next step? since the [repository](https://github.com/efeslab/DMon-AE) Tanvir gave me has no detailed information about how to use DMon.

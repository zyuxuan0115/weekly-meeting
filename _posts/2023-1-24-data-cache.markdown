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
  *-cache:2
       description: L3 cache
       physical id: 52
       slot: L3 Cache
       size: 35MiB
       capacity: 35MiB
       capabilities: synchronous internal varies unified
       configuration: level=3
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
![web-stanford](/assets/2023-01-24/web-BerkStan.png)
![web-berkstan](/assets/2023-01-24/web-Stanford.png)
![web-NotreDame](/assets/2023-01-24/web-NotreDame.png)


### intel-cmt-cat
- We later on moved to [intel-cmt-cat](https://github.com/intel/intel-cmt-cat)'s `rdtset`
- the command 
- NOTES for building `rdtset`
     + in the home directory 
```bash
> make
> sudo make install
> sudo rm /var/lock/libpqos
> pqos -s
> sudo chown -R {username} /sys/fs/resctrl/
```
- The script to run a program with a reduced cache line and a limited bandwidth is
```bash
./pagerank_lock 1 1 {workload}.txt &
PID=$!
rdtset -t 'mba=10;l3=0x1' -p $PID
```
- Why `l3=0x1`? 
     + I checked the cache info `getconf -a | grep CACHE`
```
getconf -a | grep CACHE
LEVEL1_ICACHE_SIZE                 32768
LEVEL1_ICACHE_ASSOC                8
LEVEL1_ICACHE_LINESIZE             64
LEVEL1_DCACHE_SIZE                 32768
LEVEL1_DCACHE_ASSOC                8
LEVEL1_DCACHE_LINESIZE             64
LEVEL2_CACHE_SIZE                  1048576
LEVEL2_CACHE_ASSOC                 16
LEVEL2_CACHE_LINESIZE              64
LEVEL3_CACHE_SIZE                  37486592
LEVEL3_CACHE_ASSOC                 11
LEVEL3_CACHE_LINESIZE              64
LEVEL4_CACHE_SIZE                  0
LEVEL4_CACHE_ASSOC                 0
LEVEL4_CACHE_LINESIZE              0 
```
     + It shows that  L3 cache's associtivity is 11.
     + since we want cache capacity to be 1/16th, we need `-t l3=0x1` mask.
     + however I got this error when I run the script
![error](/assets/2023-01-24/err.png)

- So finally I switch back to this script
```bash
./pagerank_lock 1 1 {workload}.txt &
PID=$!
rdtset -t 'mba=10' -p $PID
```
     + results are listed here:
![web-stanford](/assets/2023-01-24/bw-web-BerkStan.png)
![web-berkstan](/assets/2023-01-24/bw-web-Stanford.png)
![web-NotreDame](/assets/2023-01-24/bw-web-NotreDame.png)
![web-google](/assets/2023-01-24/bw-web-Google.png)
![roadNet-PA](/assets/2023-01-24/bw-roadNet-PA.png)
![roadNet-CA](/assets/2023-01-24/bw-roadNet-CA.png)

### Synthetic workloads for pagerank
![v200000d200](/assets/2023-01-24/v200000-d200.png)
![v200000d20](/assets/2023-01-24/v200000-d20.png)
- this result shows that Saba's solution may not suggest an optimal prefetch distance
     + we need to use this new method to test all workloads again

- a comparison
 
![normailzed](/assets/2023-01-24/normalized.png)

### DMon
- successfully built DMon's LLVM pass by using LLVM 7.0
	+ what is the next step? since the [repository](https://github.com/efeslab/DMon-AE) Tanvir gave me has no detailed information about how to use DMon.

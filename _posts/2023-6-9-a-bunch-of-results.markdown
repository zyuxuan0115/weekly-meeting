---
layout: post
title:  "2023-6-9 a bunch of results"
date:   2023-6-9 1:53:46 -0500
categories: data-cache
---

### What if the disp of a mov insn is an expression
- sssp 

![sssp](/assets/2023-06-09/sssp.png)

- nas-is

![is](/assets/2023-06-04/is_clang.png)

- In [BOLT code](https://github.com/upenn-acg/BOLT/blob/pg2/inject-prefetch/bolt/lib/Passes/InjectPrefetchOuterLoopPass.cpp#L748), nas-is's disp of the TopLLCMissInstr is an expression

- Objdump's outputs 
	+ sssp
		* ![s1](/assets/2023-06-09/s2.png)
	+ is
  	* ![s1](/assets/2023-06-09/s3.png)
	+ to change prefetch distance
		* we can always change the last 4 bytes of the insn before the prefetch insn

- The length of the insn changes depending on the prefetch distance
	+ sssp (prefetch distance = 0x200)
		* ![s2](/assets/2023-06-09/s2.png) 
	+ sssp (prefetch distance = 0x20)
		* ![s3](/assets/2023-06-09/s4.png)
	+ solution
		* BOLT produces a binary with a large enough prefetch distance
		* insert BOLTed function at runtime
		* without running `perf stat`, immediately change the last 4 bytes to be a very small number 

- Changed pg^2 
	+ made pg^2 work for all CRONO workloads and nas-is

### Summary of APT-GET on AJ-CGO2017

| | baseline | apt-get | swpf | bolt | 
|:---:|:---:|:---:|:---:|:---:|
|  nas-is | 1.409256 | <strong>1.007589</strong> | <strong>1.055583</strong> | <strong>1.022855</strong> |  
|  nas-cg | 5.268 | × | 5.358 |  5.479|  
|  randacc | 6.151 | × | <strong>4.263<strong>  | × |  
|  hashjoin | 14.693 | × | 14.942 | × | 
|  graph500 | 7.664 | 7.632	| <strong>7.285</strong> | 7.673 |  

#### graph-500
- g500-apt-get

![g500](/assets/2023-06-09/g500-apt-get.png) 

- g500-swpf

![g500](/assets/2023-06-09/g500_outoforder.png) 

- <strong>execution time </strong>

![g500-exec-time](/assets/2023-06-09/g500-exec-time.png)

- <strong>sw_prefetch_access.t0</strong>
	+ Number of PREFETCHT0 instructions executed

![sw-pref](/assets/2023-06-09/g500_prefetch_access.t0.png) 

- <strong>l2_rqsts.pf_hit</strong>
	+ Requests from the L1/L2/L3 hardware prefetchers or Load software prefetches that hit L2 cache

![pf-hit](/assets/2023-06-09/g500_l2_rqsts.pf_hit.png) 

- <strong>l2_rqsts.pf_miss</strong>
	+ Requests from the L1/L2/L3 hardware prefetchers or Load software prefetches that miss L2 cache

![pf-miss](/assets/2023-06-09/g500_l2_rqsts.pf_miss.png) 

#### nas-is

![is](/assets/2023-06-06/is_aptget.png)

- <strong>execution time</strong>
![is-pf-l2-hit](/assets/2023-06-06/is-exec-time.png)

- <strong>sw_prefetch_access.t0</strong>
	+ Number of PREFETCHT0 instructions executed

![is-prefetch](/assets/2023-06-06/is-prefetch_access.t0.png)

- <strong>l2_rqsts.pf_hit</strong>
	+ Requests from the L1/L2/L3 hardware prefetchers or Load software prefetches that hit L2 cache

![is-pf-l2-hit](/assets/2023-06-06/is-l2_rqsts.pf_hit.png)

- <strong>l2_rqsts.pf_miss</strong>
	+ Requests from the L1/L2/L3 hardware prefetchers or Load software prefetches that miss L2 cache

![is-pf-l2-miss](/assets/2023-06-06/is-l2_rqsts.pf_miss.png)

### multiple prefetch locations sssp

- In sssp 2 loads that have high LLC miss count
	+ `0x4015fc`
	+ `0x401611`
- both are prefetchable

![sssp_new](/assets/2023-06-09/sssp_new.png) 

- `./sssp 1 1 email-euAll 10`

| | 0x4015fc | 0x401611 | 0x4015f8 |
|:---:|:---:|:---:|:---:|
| LLC misses/sec | 2055.194662 | 1444.749882 | 57.21327729 |

| | original | prefetch 0x4015fc | prefetch 0x401611 |  
|:---:|:---:|:---:|:---:|:---:|
|  execution time | 21.318521 | 16.446077 | 20.472623 |

### multiple prefetch locations bc

- In bc 2 loads that have high LLC miss count
	+ `0x401561`
	+ `0x401565`
- both are prefetchable

![bc](/assets/2023-06-09/bc.png) 

- `./bc 1 20000 500`

| | 0x401561 | 0x401565 |
|:---:|:---:|:---:|
| LLC misses/sec | 2001.573352 | 1880.66186 |

| | original | prefetch 0x401561 | prefetch 0x401565 |  
|:---:|:---:|:---:|:---:|:---:|
|  execution time | 240.853975 | 232.816541 | 231.977601 |


### Can LLC misses/sec really tell us if a load should be prefetched?
- Case 1:
	+ input name: large_twitch_edges
	+ LLC: `0x401540`: 3215.753509 times/second
 
![large_twitch](/assets/2023-06-09/graph_large_twitch_edges.png)

- Case 2:
	+ input name: p2p-Gnutella24
	+ LLC: `0x401540`: 0.05372664389 times/second

![large_twitch](/assets/2023-06-09/graph_p2p-Gnutella24.png)
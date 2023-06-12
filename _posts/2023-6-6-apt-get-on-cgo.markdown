---
layout: post
title:  "2023-6-6 apt-get on cgo workloads"
date:   2023-6-6 1:53:46 -0500
categories: data-cache
---
### IS
- `./is`
	+ apt-get can insert prefetch
	+ BOLT can insert prefetch
	+ swpf can insert prefetch

- performance

| 10 iterations | is | isswpf(pf dist=2048) | is.bolt(pr dist=2048) | apt-get | 
|:---:|:---:|:---:|:---:|:---:|
|  exec time (seconds)  | 1.409256 | 1.055583 | 1.022855 | 1.007589 |

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

### CG
- `./cg`
	+ apt-get cannot insert prefetch to the binary
		* they do have a `.afdo` file 
		* ![s1](/assets/2023-06-06/s1.png)
	+ BOLT can insert prefetch but have no performance improvement
	+ swpf can insert prefetch but have no performance improvement

- performance

| | cg (sec) | cgswpf (sec) | cg.bolt(inner loop)| cg.bolt(outer loop)|
|:---:|:---:|:---:|:---:| :---:| 
|  1   | 5.15 | 5.67 | 5.35 | 5.34 |
|  2   | 5.43 | 5.10 | 5.71 | 5.26 |
|  3   | 5.32 | 5.32 | 5.54 | 5.48 |
|  4   | 5.46 | 5.03 | 5.56 | 5.22 |
|  5   | 5.65 | 5.32 | 5.12 | 5.97 |
|  6   | 5.00 | 5.20 | 5.70 | 5.28 |
|  7   | 5.09 | 5.72 | 5.54 | 5.70 |
|  8   | 5.04 | 5.50 | 5.41 | 5.58 |
|  <strong>average</strong> | 5.268 | 5.358 | 5.491 | 5.479 |


### randacc
- `./randacc 1000000000`
	+ apt-get cannot insert prefetch to the binary
	+ BOLT cannot insert prefetch to the binary
	+ swpf can insert prefetch to the binary, and have a significant performance improvement

- performance 

| | randacc (sec) | randaccswpf (sec) | 
|:---:|:---:|:---:|
|  1   | 6.173 | 4.403 | 
|  2   | 6.630 | 4.330 |
|  3   | 5.869 | 4.328 |
|  4   | 6.111 | 4.052 |
|  5   | 5.971 | 4.201 |
| <strong>average</strong>| 6.151 | 4.263 |

### graph-500
- `./g500 -s 18 -e 18`
	+ Top 1 LLC miss function: `make_bfs_tree`: 81.53%
	+ Top LLC miss instr: 
		* 0x40189d: 39%
		* 0x4018a8: 32%
- apt-get can insert prefetch, no performance improvement
	+ prefetch distance = 32
- BOLT can insert prefetch, no performance improvement
- swpf can insert prefetch, 4% performance improvement

| | g500 | g500 (apt-get) | g500 (swpf) | g500.bolt (outer) |
|:---:|:---:|:---:|:---:|:---:|
|  1   | 7.665 | 7.665 | 7.392 | 7.682 |
|  2   | 7.681 | 7.661 | 7.101 | 7.668 |
|  3   | 7.682 | 7.596 | 7.331 | 7.672 |
|  4   | 7.643 | 7.609 | 7.258 | 7.670 |
|  5   | 7.650 | 7.658 | 7.341 | 7.673 |
| <strong>average</strong> | 7.664 | 7.632 | 7.285 | 7.673 | 

![g500](/assets/2023-06-06/g500-apt-get.png) 

- <strong>execution time </strong>

![g500-exec-time](/assets/2023-06-06/g500-exec-time.png)

- <strong>sw_prefetch_access.t0</strong>
	+ Number of PREFETCHT0 instructions executed

![sw-pref](/assets/2023-06-06/g500_prefetch_access.t0.png) 

- <strong>l2_rqsts.pf_hit</strong>
	+ Requests from the L1/L2/L3 hardware prefetchers or Load software prefetches that hit L2 cache

![pf-hit](/assets/2023-06-06/g500_l2_rqsts.pf_hit.png) 

- <strong>l2_rqsts.pf_miss</strong>
	+ Requests from the L1/L2/L3 hardware prefetchers or Load software prefetches that miss L2 cache

![pf-miss](/assets/2023-06-06/g500_l2_rqsts.pf_miss.png) 

### hashjoin-2 & hashjoin-8
- apt-get cannot insert prefetch to `create_relation_fk`
- BOLT cannot insert prefetch to the `create_relation_fk`
	+ but BOLT can insert prefetch to `bucket_chaining_join`, `radix_cluster`, `parallel_radix_partition`
	+ `parallel_radix_partition` has 8.89% performance improvement
- swpf inserts prefetch to a wrong function, so no performence improvement
	



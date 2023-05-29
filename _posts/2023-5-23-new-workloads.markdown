---
layout: post
title:  "2023-5-23 explore new workloads"
date:   2023-5-22 7:53:46 -0500
categories: data-cache
---
The link of new workloads is [here](https://github.com/SamAinsworth/reproduce-cgo2017-paper/)

### randacc
- original

![orig](/assets/2023-05-23/randacc.png) 

- optimized

![opt](/assets/2023-05-23/randaccswpf.png)

### IS
- our solution
![is](/assets/2023-05-23/is_clang.png)

- performance

| 10 iterations | is | isswpf(pf dist=2048) | is.bolt(pr dist=2048) | 
|:---:|:---:|:---:|:---:|
|  exec time (seconds)  | 1.409256 | 1.055583 | 1.022855 |

- (1.4093-1.0556)/1.4093 = 25%


### CG
![s1](/assets/2023-05-23/s1.png)

![cg_clang](/assets/2023-05-23/cg_clang.png)

| | cg (sec) | cgswpf (sec) | cg.bolt(sec) |
|:---:|:---:|:---:|:---:| 
|  1   | 5.15 | 5.67 | 5.35 |
|  2   | 5.43 | 5.10 | 5.71 |
|  3   | 5.32 | 5.32 | 5.54 |
|  4   | 5.46 | 5.03 | 5.56 | 
|  5   | 5.65 | 5.32 | 5.12 | 
|  6   | 5.00 | 5.20 | 5.70 |
|  7   | 5.09 | 5.72 | 5.54 |
|  8   | 5.04 | 5.50 | 5.41 |


### hashjoin-ph-2 & hashjoin-ph-8
- command to run hashjoin-ph-2
	+ `./npj2epb --r-size=16777216 --s-size=268435456`

- Top LLC Miss function:
	+ ![s2](/assets/2023-05-23/s2.png)

- However, they inserted `__builtin_prefetch()` in `probe_hashtable()` function 
	+ ![s3](/assets/2023-05-23/s3.png)
	+ it even showed <strong>The perf.data data has no samples!</strong> when I ran `perf annotate --stdio -M att -i perf.data probe_hashtable`
- Because `create_relation_pk` & `create_relation_fk` are in the initialization, I checked 
	+ `radix_cluster`
	+ `bucket_chaining_join`
	+ `parallel_radix_partition`
- CFG of `create_relation_pk`

![create_relation_pk](/assets/2023-05-23/create_relation_pk.png) 

- CFG of `radix_cluster`

![radix_cluster](/assets/2023-05-23/radix_cluster.png)

- CFG of `bucket_chaining_join`
![bucket_chain_join](/assets/2023-05-23/bucket_chain_join.png)

- CFG of `parallel_radix_partition`
![parallel](/assets/2023-05-23/parallel.png)

- If we insert prefetch in `radix_cluster`, `bucket_chaining_join` & `parallel_radix_partition`
	+ How to measure performance improvement?
	+ Measure the execution time of each function?

### graph-500

![graph-500](/assets/2023-05-23/g500.png)


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

- prefetching 2 locations has the same performance as prefetching 1 location 

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

|parallel_radix_partition |  BOLTed, pf distance=128 (sec) | original(sec) |
|:---:|:---:|:---:|
|  1   | 0.547044 | 0.652046 |
|  2   | 0.659253 | 0.705690 | 
|  3   | 0.651533 | 0.715434 | 
|  4   | 0.581727 | 0.669210 |
|  5   | 0.590320 | 0.628317 | 
|  6   | 0.621251 | 0.661052 | 
|  7   | 0.641712 | 0.688747 | 
|  8   | 0.638623 | 0.690977 | 
| average|  |  |


|radix_cluster |  BOLTed, pf distance=128 (sec) | original(sec) |
|:---:|:---:|:---:|
|  1   | 0.585211 | 0.583108 |
|  2   | 0.589890 | 0.585972 | 
|  3   | 0.590394 | 0.586469 | 
|  4   | 0.583394 | 0.581534 |
|  5   | 0.592914 | 0.577221 | 
|  6   | 0.587597 | 0.579575 | 
|  7   | 0.583853 | 0.581242 | 
|  8   | 0.587139 | 0.585114 | 


|bucket_chaining_join |  BOLTed, pf distance=128 (sec) | original(sec) |
|:---:|:---:|:---:|
|  1   | 0.123915 | 0.127183 |
|  2   | 0.126259 | 0.124573 | 
|  3   | 0.125050 | 0.126704 | 
|  4   | 0.122405 | 0.124450 |
|  5   | 0.123304 | 0.123850 | 
|  6   | 0.127147 | 0.124056 | 
|  7   | 0.125577 | 0.125487 | 
|  8   | 0.124733 | 0.125009 | 



- If we insert prefetch in `radix_cluster`, `bucket_chaining_join` & `parallel_radix_partition`
	+ How to measure performance improvement?
	+ Measure the execution time of each function?

### graph-500

- `./g500-no -s 18 -e 18`

- original graph-500

![graph-500](/assets/2023-05-23/g500_no.png)

- BOLTed graph-500

![graph-500_bolt](/assets/2023-05-23/g500_bolt.png)


| | g500 | g500.bolt  | g500-man-outoforder(swpf)  |
|:---:|:---:|:---:|:---:|
|  1   | 7.642 | 8.347 | 7.315 | 

![graph-500_bolt](/assets/2023-05-23/g500_outoforder.png)



### pagerank again
![old_approach](/assets/2023-04-21/pr_orig_cfg3.png)

![pagerank](/assets/2023-05-23/pagerank_pref.png)

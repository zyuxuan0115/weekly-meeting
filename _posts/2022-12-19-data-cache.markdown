---
layout: post
title:  "2022-12-19 data cache"
date:   2022-12-13 10:53:46 -0500
categories: data-cache 
---

|    pagerank<br> 1000 iterations   | prefetch dist | execution time <br> original | execution time <br> optimized | average<br> speedup |
| :----:        |    :----:							|   :----:			| :----: |  :----: |  :----: |  :----: |
| <strong>web-Google</strong><br>Vertices:916428<br> Degree:456   |   32   | 28.596261(32)<br>28.825631(32)<br>29.023874(32)<br>28.744529(32)<br>29.537645(32) | 21.337055(32)<br>20.768182(32)<br>21.362526(32)<br>20.829610(32)<br>21.191370(32) | 27.112% |
| <strong>web-BerkStan</strong><br>Vertices:685231<br> Degree:249   |   242<br>236x2<br>231<br>238   | 14.705026(242)<br>14.272114(236)<br>14.425358(231)<br>14.531888(236)<br>14.677635(238) | 8.882512(242)<br>9.076098(236)<br>8.808531(231)<br>8.811421(236)<br>9.073147(238) | 38.505% |
| <strong>web-BerkStan</strong><br>optimized binary <br>from web-Google  |   32     |  | 8.834033(32)<br>8.888177(32)<br>8.880781(32)<br>8.925810(32)<br>8.972979(32) | 38.712% |
| <strong>web-BerkStan</strong><br>([20:],[5,15])  | 166<br>134<br>123<br>6<br>7 | 14.507541(166)<br>14.983151(134)<br>14.999300(123)<br>14.400081(6)<br>14.345355(7)     | 9.057653(166)<br>9.077510(134)<br>8.956206(123)<br>10.673848(6)<br>10.581121(7) | 39.107%(120+)<br>26.058%(10-) | 


The execution time for producing an optimized binary (`perf record` time (20s) is included):

|      | real | user | system | 
| :----: |:----:|   :----:| :----: | 
| web-Google  |  3m15.850s| 3m26.829s | 2m40.915s | 
| web-BerkStan | 3m12.146s | 3m23.843s | 2m38.148s |
| web-Stanford | 2m43.879s | 3m0.540s | 2m43.761s |
| web-NotreDame| 3m8.183s | 3m11.062s | 2m51.289s |
| roadNet-PA | 3m31.861s | 3m47.807s | 2m41.504s |
| roadNet-CA | 3m43.796s | 3m58.652s | 2m42.920s |


### Discussion with Saba
- Why `peakidx = signal.find_peaks_cwt(col2_ar[20:],  np.arange(5,15), noise_perc=0.1)`
    + Because of [this](https://stackoverflow.com/questions/25571260/scipy-signal-find-peaks-cwt-not-finding-the-peaks-accurately).
    + The slice starting from [20:] is because Saba thought this can delete some noise of the data on her machine. (But this may be different on our machine.)
- Why always prefetch dist = 32?
    + Because of [this](https://github.com/upenn-acg/floar/blob/master/apt-get/python-codes/find-peaks.py#L135)
- Can we randomly change the prefetch distance suggested by [Saba's python script](https://github.com/upenn-acg/floar/blob/master/apt-get/scripts/capture_PCs_real.sh#L216) when producing the optimal binary (the binary tha has prefetch inserted)?
    + Yes, we can.
    + The `-input-file` only contains the prefetch location


### Test which is the optimal prefetch distance
![fake](/assets/2022-12-19/berkstan_fake.png)

- the <strong>pagerank-INPUT-web-Stanford-prefetch.afdo</strong> file
    + contains the function that need to insert a prefetch insn
    + contains the location for the prefetch to be inserted 
    + contains the <strong>prefetch distance</strong>.
```
_Z7do_workPv:1:0
 68: 0 __prefetch_nta_0:57000
```
- the code for autofdo's output's data format is [here](https://github.com/google/autofdo/blob/master/profile_writer.h#L50)
- I noticed that the prefetch is proportional to the number at the end of the second line
    + so when I change the `$dist`, I also change this number in the the `.afdo` file.

![fake](/assets/2022-12-19/berkstan.png)
![fake](/assets/2022-12-19/stanford.png)
![fake](/assets/2022-12-19/google.png)

### OGB workloads
[OGB workloads](https://ogb.stanford.edu/docs/lsc/), the way to acquire these data is in this [script](https://github.com/upenn-acg/floar/blob/master/apt-get/inputs2/dataloader.py)

- perf record result of workloads provided by apt-get's script
    + `perf record --freq=max -e cpu/event=0xd1,umask=0x20,name=MEM_LOAD_RETIRED.L3_MISS/ppp -- <benchmark>`
    + `perf report --stdio > pagerank-INPUT-roadmap-CA-perfReport.txt"`
```
# To display the perf.data header info, please use --header/--header-only options.
#
#
# Total Lost Samples: 0
#
# Samples: 323K of event 'MEM_LOAD_RETIRED.L3_MISS'
# Event count (approx.): 1975603470
#
# Overhead  Command   Shared Object     Symbol
# ........  ........  ................  .......................
#
    99.92%  pagerank  pagerank          [.] do_work
     0.04%  pagerank  pagerank          [.] main
     0.02%  pagerank  [unknown]         [k] 0xffffffffb945e13a
     0.00%  pagerank  [unknown]         [k] 0xffffffffb9c7649e
     0.00%  pagerank  [unknown]         [k] 0xffffffffb946e830
     0.00%  pagerank  [unknown]         [k] 0xffffffffb945e395
     0.00%  pagerank  [unknown]         [k] 0xffffffffb92d64a5
     0.00%  pagerank  [unknown]         [k] 0xffffffffb9327ece
     0.00%  pagerank  [unknown]         [k] 0xffffffffb932c3b9
     0.00%  pagerank  [unknown]         [k] 0xffffffffb9339af5
     0.00%  pagerank  [unknown]         [k] 0xffffffffb941399a
     0.00%  pagerank  [unknown]         [k] 0xffffffffb94b1043
     0.00%  pagerank  [unknown]         [k] 0xffffffffb94b37f0
     0.00%  pagerank  [unknown]         [k] 0xffffffffb945e387
     0.00%  pagerank  [unknown]         [k] 0xffffffffb944ee75
     0.00%  pagerank  [unknown]         [k] 0xffffffffb9436705
     0.00%  pagerank  [unknown]         [k] 0xffffffffb931b7e4
```

However, when we ran OGB's workloads as input
```
# To display the perf.data header info, please use --header/--header-only options.
#
#
# Total Lost Samples: 0
#
# Samples: 7K of event 'MEM_LOAD_RETIRED.L3_MISS'
# Event count (approx.): 405702
#
# Overhead  Command   Shared Object     Symbol
# ........  ........  ................  ......................
#
    75.49%  pagerank  [unknown]         [k] 0xffffffffb946e830
     9.23%  pagerank  [unknown]         [k] 0xffffffffb9c7649e
     7.53%  pagerank  [unknown]         [k] 0xffffffffb9c73862
     1.36%  pagerank  [unknown]         [k] 0xffffffffb9c73844
     0.87%  pagerank  [unknown]         [k] 0xffffffffb946e79e
     0.53%  pagerank  [unknown]         [k] 0xffffffffb9c73ac9
     0.49%  pagerank  [unknown]         [k] 0xffffffffb944ee75
     0.48%  pagerank  [unknown]         [k] 0xffffffffb9447a24
     0.36%  pagerank  [unknown]         [k] 0xffffffffb92733c8
     0.31%  pagerank  [unknown]         [k] 0xffffffffb945e13a
     0.22%  pagerank  [unknown]         [k] 0xffffffffb92d90b0
     0.21%  pagerank  [unknown]         [k] 0xffffffffb92d908d
     0.14%  pagerank  [unknown]         [k] 0xffffffffb92d909a
     0.14%  pagerank  [unknown]         [k] 0xffffffffb94467eb
     0.13%  pagerank  [unknown]         [k] 0xffffffffb92d90b7
     0.12%  pagerank  [unknown]         [k] 0xffffffffb92d90a4
     0.12%  pagerank  [unknown]         [k] 0xffffffffb94d7dd1
     0.11%  pagerank  [unknown]         [k] 0xffffffffb94467e7
     0.11%  pagerank  [unknown]         [k] 0xffffffffb9743db9
     0.10%  pagerank  [unknown]         [k] 0xffffffffb94d9d53
     0.10%  pagerank  [unknown]         [k] 0xffffffffb945e0f7
```

They are both inputs feed into `pagerank`, why are the top LLC miss functions different?

My guess: 
The `OGB` workload is too large, so when `perf record` collect profiles, the `pagerank` is still running on the initialization of graph. 

How long is the `OGB` inputs initialized? I added [code](https://github.com/upenn-acg/floar/blob/master/apt-get/CRONO/apps/pagerank/pagerank_lock.cc#L150) to measure it.



The input provided from apt-get's script
```
benchmark_name:   pagerank
input-graph:      roadNet-CA.txt
################################## LLC misses
Capture deliquent load PCs ...
  0) run the benchmark to record the running time ...

.gr graph with parameters: Vertices:1971281 Degree:12 LinesInFile:4
Memory Initialized
File Read
Largest Vertex: 1971281
Initialization Done
Initialization Time:3.312455 seconds
do_work is executing!

Time:39.300727 seconds
```

```shell
zyuxuan@acgcascade52:~/floar/apt-get$ ./CRONO/apps/pagerank/pagerank_lock 1 1 inputs2/author_paper.txt
Killed
zyuxuan@acgcascade52:~/floar/apt-get$ ./CRONO/apps/pagerank/pagerank_lock 1 1 inputs2/author_paper.txt
Killed
```
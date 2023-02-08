---
layout: post
title:  "2023-2-2 data cache"
date:   2023-2-2 8:53:46 -0500
categories: data-cache 
---
### bc's results 
- results of synthetic inputs
![bc-syn](/assets/2023-02-02/bc-syn.png)
- results of real inputs

|      | web-Google | web-BerkStan | web-Stanford | 
| :----: | :----:| :----:| :----: | 
| execution time (second) | 0.00004 |  0.000039 | 0.000043 | 

| | web-NotreDame | roadNet-PA | roadNet-CA |
| :----: | :----: | :----: | :----: | 
| execution time (second)| 0.000039 | 0.000039 |  0.000039 | 

![bc-real1](/assets/2023-02-02/bc-roadNet-PA.png)
![bc-real2](/assets/2023-02-02/bc-roadNet-CA.png)
![bc-real3](/assets/2023-02-02/bc-web-Google.png)
![bc-real4](/assets/2023-02-02/bc-web-Stanford.png)
![bc-real5](/assets/2023-02-02/bc-web-BerkStan.png)
![bc-real6](/assets/2023-02-02/bc-web-NotreDame.png)
### pr's results on haswellcat16

![web-Google](/assets/2023-02-02/web-Google.png)
![web-Stanford](/assets/2023-02-02/web-Stanford.png)
![web-BerkStan](/assets/2023-02-02/web-BerkStan.png)
![web-NotreDame](/assets/2023-02-02/web-NotreDame.png)
![web-roadNet-CA](/assets/2023-02-02/roadNet-CA.png)
![web-roadNet-PA](/assets/2023-02-02/roadNet-PA.png)

### dfs synthetic inputs
`dfs`'s afdo file (produced by `autofdo`)
`dfs` contains 4 locations that need to be prefetched

```
_Z7do_workPv:1:0
 63: 0 __prefetch_nta_0:131000
 65478: 0 __prefetch_nta_0:32000
_Z12init_weightsiiPPiS0_:1:0
 13: 0 __prefetch_nta_0:32000
 54: 0 __prefetch_nta_0:32000
 ```

![dfs1](/assets/2023-02-02/dfs1.png)
![dfs2](/assets/2023-02-02/dfs2.png)
![dfs3](/assets/2023-02-02/dfs3.png)
![dfs4](/assets/2023-02-02/dfs4.png)
![dfs5](/assets/2023-02-02/dfs5.png)
![dfs6](/assets/2023-02-02/dfs6.png)
![dfs7](/assets/2023-02-02/dfs7.png)

### bfs's results
- results of real inputs
![bfs-real](/assets/2023-02-02/bfs-real.png)

- results of synthetic inputs
![bfs-syn](/assets/2023-02-02/bfs-syn.png)

### sssp's results
`sssp` contains 2 locations that need to be prefetched

```
_Z7do_workPv:1:0
 57: 0 __prefetch_nta_0:32000
 65: 0 __prefetch_nta_0:32000
```

another fact of `sssp`

| exe time (sec)	| optimized-57	| optimized-65 | original(clang)| original(gcc) |
| :----: | :----:| :----:| :----: |   :----: | 
| v=30000,d=350 |	4.05298 | 4.076476 | 25.919367 | 4.074955 |
| v=30000,d=1000 |	9.85139	| 9.753668 | 10.796108 | 22.436999 |

![sssp1](/assets/2023-02-02/sssp1.png)
![sssp4](/assets/2023-02-02/sssp4.png)
![sssp5](/assets/2023-02-02/sssp5.png)
![sssp6](/assets/2023-02-02/sssp6.png)

### discussion
- In apt-get's script it produces the optimized binary in this way:
```bash
$AutoFDO10/create_llvm_prof --binary=$benchmark_name --profile=$benchmark_name"-INPUT-"$g"-ALL-dist2.csv" --profiler="prefetch" --format=text --out=$benchmark_name"-INPUT-"$g"-prefetch.afdo"
$LLVM10_buildMyPasses/bin/opt -load $LLVM10_buildMyPasses"/lib/SWPrefetchingLLVMPass.so" -S -SWPrefetchingLLVMPass -input-file $benchmark_name"-INPUT-"$g"-prefetch.afdo" -dist $dist  <$benchmark_name".ll"> $benchmark_name"-pref-INPUT-"$g".ll"
$LLVM10_buildMyPasses/bin/clang  --std=c++0x -O3  -fdebug-info-for-profiling  -Wall -Werror   $benchmark_name"-pref-INPUT-"$g".ll" -c
$LLVM10_buildMyPasses/bin/clang -g --std=c++0x -O3  -fdebug-info-for-profiling -Wall -Werror  $benchmark_name"-pref-INPUT-"$g".o" -o $benchmark_name"-pref-INPUT-"$g -lpthread -lrt
```
- pageranks *.afdo file only contains the following info:
```
_Z7do_workPv:1:0
 68: 0 __prefetch_nta_0:57000
```
- In google autofdo's [source code](https://github.com/google/autofdo/blob/master/profile_writer.h), they explained the format of *.afdo file
- also, in [apt-get's code](https://github.com/upenn-acg/floar/blob/master/apt-get/SWPrefetchingLLVMPass/SWPrefetchingLLVMPass.cpp#L652), they divided 57000 by 1000.

### limit bandwidth
- Because `bfs` on real inputs's normalized speedup is close to 1
![bfs-real](/assets/2023-02-02/bfs-real.png)
    + we decided to limit the bandwidth of `bfs` with `loc-brightkite_edges`
- To measure the bandwidth, I used `sudo pqos -I -p 'mbl:pids;mbr:pids'` from [intel-cmt-cat](https://github.com/intel/intel-cmt-cat/wiki/MBM-MBA-how-to-guide#mba---mbm---configure-mba-in-mbs)
- The script is this
```bash
./bfs 1 1 loc-brightkite_edges.txt &
PID=$!
pqos -I -p "mbl:${PID};mbr:${PID}"
wait
```
![bw](/assets/2023-02-02/bandwidth.png)
- The max bandwidth of our machine is <strong>131.13 GiB/s</strong>
    + 20/(131.13*1000) = 0.015% (of the max bandwidth)
- I choose 20 MiB/s as the bandwidth limitation.
    + `rdtset` support [setting max bandwidth](https://github.com/intel/intel-cmt-cat/tree/master/rdtset)
        * b, mba_max for max allowable local memory B/W
        * `rdtsed -t 'mba_max=2000;cpu=1`
    + so the command I used is 
```bash
taskset --cpu-list 1 ./bfs 1 1 loc-brightkite_edges.txt &
PID=$!
rdtsed -t 'mba_max=20;cpu=1' -p ${PID}
wait
```
    + I got the following error message
```
Allocation: MBA CTRL requested but not enabled. Please enable MBA CTRL.
Requested configuration is not valid!
Allocation: Failed to configure allocation!
```
    + how to change the enable MBA CTRL?
        * this webpage has some [info](https://github.com/intel/intel-cmt-cat/wiki/MBM-MBA-how-to-guide#mba---mbm---configure-mba-in-mbs)
        * the command to enable MBA CTRL is `sudo pqos -I -R mbaCtrl-on`
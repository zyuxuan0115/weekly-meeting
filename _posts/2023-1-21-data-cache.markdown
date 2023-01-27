---
layout: post
title:  "2023-1-21 data cache"
date:   2023-01-21 10:53:46 -0500
categories: data-cache 
---
### Can we add the number of iterations in pagerank?
- the execution time of pagerank workloads are too short
- to enable online code replacement, we need a longer execution time
- however, simply adding the number of iteration might make these workloads to be too fake?
- in practice, how many iterations does 

### performance of the optimized pagerank with different prefetch distance
![web-google](/assets/2023-01-21/web-Google.png) 
![web-stanford](/assets/2023-01-21/web-stanford.png) 
![web-BerkStan](/assets/2023-01-21/web-BerkStan.png) 
![web-NotreDame](/assets/2023-01-21/web-NotreDame.png) 
![roadNet-PA](/assets/2023-01-21/roadNet-PA.png) 
![roadNet-CA](/assets/2023-01-21/roadNet-CA.png) 

### The execution time for producing an optimized binary 
- In the following table's execution time, the time spend on `perf record` (20s) is included:

|      | real | user | system | 
| :----: |:----:|   :----:| :----: | 
| web-Google  |  3m15.850s| 3m26.829s | 2m40.915s | 
| web-BerkStan | 3m12.146s | 3m23.843s | 2m38.148s |
| web-Stanford | 2m43.879s | 3m0.540s | 2m43.761s |
| web-NotreDame| 3m8.183s | 3m11.062s | 2m51.289s |
| roadNet-PA | 3m31.861s | 3m47.807s | 2m41.504s |
| roadNet-CA | 3m43.796s | 3m58.652s | 2m42.920s |

### The execution time of bc

|    bc   | prefetch dist | execution time <br> original | execution time <br> optimized | average<br> speedup | 
| :----:        |    :----:							|   :----:			| :----: | :----: |  :----: |
| Vertices:56384<br> Degree:8  |     32x5<br>2004<br>1951   | 78.197884(32)<br>77.895855(32)<br>78.074177(32)<br>78.920343(32)<br>81.451345(32)<br>79.119448(2004)<br>79.795526(1951) | 67.574543(32)<br>67.412499(32)<br>67.127531(32)<br>67.846038(32)<br>69.158621(32)<br>78.382337(2004)<br>74.711875(1951) | 14.047%(32)<br>3.663%(2000) |
| Vertices:40000<br> Degree:10  |   32   | 47.212300(32)<br>47.383155(32)<br>47.279665(1185)<br>48.478178(32)<br>47.527180(32) | 42.218670(32)<br>42.056944(32)<br>44.820932(1185)<br>42.195524(32)<br>41.896053(32) | 11.665%(32)<br>5.2%(1185) |
| Vertices:20000<br> Degree:50  |    32   | 24.151465(32)<br>24.010671(32)<br>24.214636(32)<br>24.009164(32)<br>24.003759(642) | 23.262613(32)<br>23.017122(32)<br>23.164027(32)<br>22.992478(32)<br>23.169104(642)| 4.098%(32)<br>3.477%(642) |
| Vertices:20000<br> Degree:100  |   467<br>456<br>506<br>443<br>483   |41.527720(467)<br>41.399529(456)<br>41.753334(506)<br>41.473342(443)<br>41.677791(483) | 72.221025(467)<br>71.523556(456)<br>71.569752(506)<br>71.800227(443)<br>71.644434(483)| -72.619% |
 | Vertices:20000<br> Degree:200  |  32   |86.797034(32)<br>85.867522(32)<br>86.136966(32)<br>86.431068(32)<br>86.942206(32) | 121.898140(32)<br>122.466065(32)<br>133.931805(32)<br>124.171229(32)<br>124.245491(32)| -45.014% |
| Vertices:10000<br> Degree:500  |   371<br>466<br>415<br>443<br>450<br>32x2   | 50.337(371)<br>50.283521(466)<br>50.007282(415)<br>49.613216(443)<br>49.976438(450)<br>49.980702(32)<br>49.825(32) | 89.602(371)<br>88.117181(466)<br>89.360649(415)<br>91.218368(443)<br>89.325455(450)<br>73.102367(32)<br>72.573(32) |  -78.893%(400)<br>-45.959%(32) |

### issues with building DMon
- I used llvm version 16.0 to build DMon's `selective prefetch` pass
```bash
cp path/to/dmon/llvm-passes/selective-prefetch ../llvm/lib/Transforms
vim ../llvm/lib/Transforms/CMakeLists.txt 
```
- at the end of the file, add 
```
add_subdirectory(selective-prefetch)
```
- but I got the following error
![dmon-error](/assets/2023-01-21/dmon-error.png)

- my guess is the version of llvm is wrong. 
    + so which version of llvm you were using?

- After building the selective prefetch pass ...
    + how to use it?
    ```
    clang -emit-llvm -S test1.c
    opt -load ../build/lib/selective-prefetch.so < test1.ll > /dev/null
    ```



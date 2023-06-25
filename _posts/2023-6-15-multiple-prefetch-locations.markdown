---
layout: post
title:  "2023-6-15 multiple-prefetch-locations"
date:   2023-6-15 1:53:46 -0500
categories: data-cache
---

### prefetch for multiple locations

- sssp
	+ `./sssp 1 1 email-euAll 10`
	+ prefetch distance for both locations: 64

| | 0x4015fc | 0x401611 | 0x4015f8 |
|:---:|:---:|:---:|:---:|
| LLC misses/sec | 2055.194662 | 1444.749882 | 57.21327729 |

| | original | prefetch both | prefetch 0x4015fc | prefetch 0x401611 | 
|:---:|:---:|:---:|:---:|:---:|
| execution time <br> (second) | 20.945770 |  <strong>12.749177</strong> | 15.837548 | 19.714539 | 


- bc
  + `./bc 1 20000 500` 

| | 0x401561 | 0x401565 |
|:---:|:---:|:---:|
| LLC misses/sec | 2001.573352 | 1880.66186 | 
 
| | original | prefetch both |  prefetch 0x401561 | prefetch 0x401565 |
|:---:|:---:|:---:|:---:|:---:|
| execution time <br> (second) | 249.047044 | <strong>274.041806</strong> | 228.323513 |  238.846107 | 


### prefetch without bounds check

| | randacc | randacc.bolt | randaccswpf |
|:---:|:---:|:---:|:---:|
| 1 | 6.116 | 5.514 | 4.305 | 
| 2 | 6.676 | 5.555 | 3.371 | 
| 3 | 6.308 | 5.780 | 3.338 | 
| 4 | 6.719 | 5.589 | 4.320 | 
| 5 | 6.195 | 4.356 | 4.195 | 
| average | 6.4028 | 5.3588 | 3.9085 | 


### The story about how to prefetch for indirect memory access
- a[i];
- b[f(a[i])];
- f(x)
	+ f(x) = 3x+1
	+ f(x) = 
![randacc](/assets/2023-06-15/rand.png)






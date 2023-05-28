---
layout: post
title:  "2023-5-23 explore new workloads"
date:   2023-5-22 7:53:46 -0500
categories: data-cache
---
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








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
- original

![orig1](/assets/2023-05-23/is.png) 

- optimized

![opt](/assets/2023-05-23/isswpf.png)

| 10 iterations | is | isswpf |
|:---:|:---:|:---:| 
|  exec time (seconds)  | 1.409256| 1.055583 |

- (1.4093-1.0556)/1.4093 = 25%


- comparison

![is_3](/assets/2023-05-23/is_3.png)

- our solution
![is_4](/assets/2023-05-23/is_4.png)


### CG
![s1](/assets/2023-05-23/s1.png)

![cg_clang](/assets/2023-05-23/cg_clang.png)

| | cg (sec) | cgswpf (sec) |
|:---:|:---:|:---:| 
|  1   | 5.15 | 5.67 |
|  2   | 5.43 | 5.10 |
|  3   | 5.32 | 5.32 |
|  4   | 5.46 | 5.03 |
|  5   | 5.65 | 5.32 |
|  6   | 5.00 | 5.20 |
|  7   | 5.09 | 5.72 |
|  8   | 5.04 | 5.50 |








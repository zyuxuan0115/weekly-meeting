---
layout: post
title:  "2023-10-23 nightcore performance"
date:   2023-10-23 1:53:46 -0500
categories: serverless functions
---
### More about [nightcore](https://www.cs.utexas.edu/users/witchel/pubs/jia21asplos-nightcore.pdf)
- OS pipe + shared memory to pass function arguments and message
	+ if the length is short, OS pipe for data transfer
	+ if the length is long, OS pipe for communication, shared memory for data transfer
- nightcore 
	+ ![s3](/assets/2023-10-24/s3.png)
- 2 microservice workloads are written in C++
	+ if we use compiler to merge functions and are to use nightcore workloads, we can use these 2 
- ![microservice](/assets/2023-10-24/s2.png) 

### Intel MPK
- Intel Memory Protection Keys (Intel MPK)
- [paper](https://www.usenix.org/system/files/atc19-park-soyeon.pdf)
	+ ![per-thread-usage](/assets/2023-10-24/s1.png)

### call invocation time on cloudlab

| microseconds  | nightcore internal | pthread | normal | 
| :----: | :----:| :----:| :----: | 
| 50th tail latency |  44.5 | 181 | 1 | 

- pthread: execution time from `pthread_create()` to `pthread_join()`

- since the overhead of `pthread_create()` is high
	+ first create the threads. 
	+ after the caller function calls, let the thread execute 

| microseconds  | nightcore internal | pthread create | pthread lock <br> as signal | normal | 
| :----: | :----:| :----:|:----:| :----:| 
| 50th tail latency |  44.5 | 181 | 22 | 1 | 

![s4](/assets/2023-10-24/s4.png)


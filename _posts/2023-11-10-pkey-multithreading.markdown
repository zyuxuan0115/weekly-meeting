---
layout: post
title:  "2023-11-03 vHive & libmpk"
date:   2023-11-03 1:53:46 -0500
categories: serverless functions
---
### pthread execution time
- theoretically the overhead of `pthread_mutex_lock()`
	+ 40-50 ns
	+ reality: 2200 ns (2.2 microseconds) 

```c++
auto start = high_resolution_clock::now();
pthread_mutex_lock(&mutex);
auto end = high_resolution_clock::now();
auto duration = duration_cast<nanoseconds>(end - start);
```

- This is the case when lock is free. 
- When there are more than 1 threads, contention for the lock may exist
	+ 22 microseconds



### more about pkey (Intel MPK)
- [how to use pkey](https://www.kernel.org/doc/html/next/core-api/protection-keys.html)
- [pthread_key_create()](https://linux.die.net/man/3/pthread_key_create)

### where the call stacks are in multithread programs

### if nightcore transfer huge amount of data
- how nightcore internal call works
	+ [Engine::OnRecvMessage()](https://github.com/ut-osa/nightcore/blob/asplos-release/src/engine/engine.cpp#L223) 

### measure vhive's function invocation time

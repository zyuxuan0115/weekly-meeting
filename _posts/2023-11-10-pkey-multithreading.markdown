---
layout: post
title:  "2023-11-03 multithread invocation time & pkey"
date:   2023-11-03 1:53:46 -0500
categories: serverless functions
---
### pthread execution time
- theoretically the overhead of `pthread_mutex_lock()`, when the lock is free
	+ 40-50 ns
	+ reality: 2000 ns (2 microseconds) 

- Sebastian: a faster way to use signal between threads
	+ [link here](https://stackoverflow.com/questions/4016789/sleeping-in-a-thread-c-posix-threads/4676069#4676069)

```
Time spent on acquiring a free lock: 2092 ns
Time spent on pthread (mutex): 18915 ns
Time spent on pthread (CondVar): 20363 ns
Time spent on pthread (signal): 23226 ns
Time spent on normal func call: 164 ns
```

- Kostas: conditional variable to take no longer than 5 microseconds
	+ pin the 2 threads to different cores
	+ performance even worse

```
Time spent on acquiring a free lock: 1644 ns
Time spent on pthread (mutex): 18437 ns
Time spent on pthread (CondVar): 60438 ns
Time spent on pthread (signal): 66536 ns
Time spent on normal func call: 121 ns
```

### how does nightcore internal's OS pipe work

### more about pkey (Intel MPK)
- [how to use pkey](https://www.kernel.org/doc/html/next/core-api/protection-keys.html)
- [pthread_key_create()](https://linux.die.net/man/3/pthread_key_create)

### where the call stacks are in multithread programs

### if nightcore transfer huge amount of data
- how nightcore internal call works
	+ [Engine::OnRecvMessage()](https://github.com/ut-osa/nightcore/blob/asplos-release/src/engine/engine.cpp#L223) 

### measure vhive's function invocation time

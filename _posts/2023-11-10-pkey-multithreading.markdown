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
	+ performance <strong>even worse</strong>

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
	+ Before a pkey can be used, it must first be allocated with `pkey_alloc()`. 
	+ An application calls the WRPKRU instruction directly in order to change access permissions to memory covered with a key. 
	+ In this example WRPKRU is wrapped by a C function called `pkey_set()`.

```c++
pkey = pkey_alloc(0, PKEY_DISABLE_WRITE);
ptr = mmap(NULL, PAGE_SIZE, PROT_NONE, MAP_ANONYMOUS|MAP_PRIVATE, -1, 0);
ret = pkey_mprotect(ptr, PAGE_SIZE, PROT_READ|PROT_WRITE, pkey);
// ... application runs here
```

- more explaination about pkey
	+ [https://man7.org/linux/man-pages/man7/pkeys.7.html](https://man7.org/linux/man-pages/man7/pkeys.7.html)
  + [https://www.gnu.org/software/libc/manual/html_node/Memory-Protection.html#Memory-Protection-Keys](https://www.gnu.org/software/libc/manual/html_node/Memory-Protection.html#Memory-Protection-Keys)
	+ how to check if hardware support Intel MPK
		* [https://www.linuxhowtos.org/manpages/7/pkeys.htm](https://www.linuxhowtos.org/manpages/7/pkeys.htm)
	+ how to use pkey in GO: [here](https://charlycst.github.io/posts/mpk/)

- to choose hardware on cloudlab [here](https://docs.cloudlab.us/hardware.html)

- [pthread_key_create()](https://linux.die.net/man/3/pthread_key_create)


### where the call stacks are in multithread programs

### if nightcore transfer huge amount of data
- how nightcore internal call works
	+ [Engine::OnRecvMessage()](https://github.com/ut-osa/nightcore/blob/asplos-release/src/engine/engine.cpp#L223) 

### measure vhive's function invocation time

### chat with Tanvir
- a [paper](https://homes.cs.washington.edu/~arvind/papers/google-rpc.pdf) suggested from Tanvir

---
layout: post
title:  "2023-11-10 multithread invocation time & pkey"
date:   2023-11-10 1:53:46 -0500
categories: serverless functions
---
### pthread execution time
- theoretically the overhead of `pthread_mutex_lock()`, when the lock is free
	+ 40-50 ns
	+ reality: 2000 ns (2 microseconds) 

- <strong>Sebastian</strong>: a faster way to use signal between threads
	+ [link here](https://stackoverflow.com/questions/4016789/sleeping-in-a-thread-c-posix-threads/4676069#4676069)

```
Time spent on acquiring a free lock: 2092 ns
Time spent on pthread (mutex): 18915 ns
Time spent on pthread (CondVar): 20363 ns
Time spent on pthread (signal): 23226 ns
Time spent on normal func call: 164 ns
```

- <strong>Kostas</strong>: conditional variable to take no longer than 5 microseconds
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
![s2](/assets/2023-11-10/s2.png)

- how nightcore internal call works
	+ launcher writes message by calling [EngineConnection::WriteMessage()](https://github.com/zyuxuan0115/nightcore/blob/asplos-release/src/launcher/engine_connection.cpp#L73)
		* in launcher's [EngineConnection::start()](https://github.com/zyuxuan0115/nightcore/blob/asplos-release/src/launcher/engine_connection.cpp#L23), event loop `uv_loop` is initialized to be either `pipe_handle` or `tcp_handle` 
		* [uv_write()](https://github.com/zyuxuan0115/nightcore/blob/asplos-release/src/launcher/engine_connection.cpp#L82) will write to pipe. 
	+ engine receives message by [Engine::OnRecvMessage()](https://github.com/ut-osa/nightcore/blob/asplos-release/src/engine/engine.cpp#L223)
		* this function checks whether the payload uses pipe or shared memory
		* the only caller is [UV_READ_CB_FOR_CLASS()](https://github.com/ut-osa/nightcore/blob/asplos-release/src/engine/message_connection.cpp#L276)
			* a little bit about [lambda](https://www.cprogramming.com/c++11/c++11-lambda-closures.html)
			* a little bit about [callback functions](https://developer.mozilla.org/en-US/docs/Glossary/Callback_function)
			* `UV_READ_CB_FOR_CLASS()` is defined [here](https://github.com/ut-osa/nightcore/blob/asplos-release/src/common/uv.h#L101)
- nightcore creates pipe using [libuv](https://github.com/libuv/libuv).
	+ from [uv.h](https://github.com/libuv/libuv/blob/v1.x/include/uv.h#L821), we know 
		* `uv_pipe_t` on Unix is a [Unix domain socket](https://en.wikipedia.org/wiki/Unix_domain_socket).
	+ corresponding [uv.c](https://github.com/libuv/libuv/blob/v1.x/src/unix/pipe.c)

### if nightcore transfers huge amount of data
![latency](/assets/2023-11-10/latency.png)

- Nightcore’s message channels are designed for low-latency message passing between its engine and other components, which carry fixed-size 1KB messages. 
	+ the first 64 bytes of a message is the header which contains the message type
and other metadata
	+ the remaining 960 bytes are message payload. 


### more about pkey (Intel MPK)
- [how to use pkey](https://www.kernel.org/doc/html/next/core-api/protection-keys.html)
	+ Before a pkey can be used, it must first be allocated with `pkey_alloc()`. 
	+ An application calls the WRPKRU instruction directly in order to change access permissions to memory covered with a key. 
	+ In this example WRPKRU is wrapped by a C function called `pkey_set()`.

```c++
int pkey = pkey_alloc(0, PKEY_DISABLE_WRITE);
void* ptr = mmap(NULL, PAGE_SIZE, PROT_NONE, MAP_ANONYMOUS|MAP_PRIVATE, -1, 0);
int ret = pkey_mprotect(ptr, PAGE_SIZE, PROT_READ|PROT_WRITE, pkey);
// ... application runs here
```

- if the application needs to update the data at `ptr`, it can gain access, do the update, then remove its write access:

```c++
pkey_set(pkey, 0); // clear PKEY_DISABLE_WRITE
*ptr = foo; // assign something
pkey_set(pkey, PKEY_DISABLE_WRITE); // set PKEY_DISABLE_WRITE again
```

- more explainations about pkey
	+ [https://man7.org/linux/man-pages/man7/pkeys.7.html](https://man7.org/linux/man-pages/man7/pkeys.7.html)
  + [https://www.gnu.org/software/libc/manual/html_node/Memory-Protection.html#Memory-Protection-Keys](https://www.gnu.org/software/libc/manual/html_node/Memory-Protection.html#Memory-Protection-Keys)
- <strong>how to use pkey in GO</strong>: [here](https://charlycst.github.io/posts/mpk/)

### hardware support for Intel MPK
- I ran the following code

```c++
int pkey = pkey_alloc(0, PKEY_DISABLE_ACCESS);
if (pkey<0){
   printf("pkey = %d, no available keys\n", pkey);
   err(EXIT_FAILURE, "pkey_alloc");
}
```

- and got the result

```c++
pkey = -1, no available keys
pkey_test: pkey_alloc: Invalid argument
```

- from this [doc](https://www.kernel.org/doc/html/next/core-api/protection-keys.html), only <strong>Intel Skylake</strong> (server CPU) and after, support pkey.
	+ how to check if hardware support Intel MPK
		* [https://www.linuxhowtos.org/manpages/7/pkeys.htm](https://www.linuxhowtos.org/manpages/7/pkeys.htm)
		* `/proc/cpuinfo` under the `flags` field. 
			- The string `pku` in this field indicates hardware support for protection keys 
			- The string `ospke` indicates that the kernel contains and has enabled protection keys support.
		* a [list](https://en.wikipedia.org/wiki/List_of_Intel_Xeon_processors_(Skylake-based)) of skylake based Intel processors
	+ The current processor I'm using doesn't support pkey
		* looked into cloudlab's documentation to choose hardware on cloudlab [here](https://docs.cloudlab.us/hardware.html) that support pkey
	+ Tried <strong>Intel skylake</strong> processor <strong>c6420</strong> and profile <strong>vhive-ubuntu20</strong>
		* it supports pkey.
- problem with Intel MPK and pkey
	* the key is too simple
	* the values of newly allocated pkeys are 1,2,3,...,15

- Other isolation approach
	+ [pthread_key_create()](https://linux.die.net/man/3/pthread_key_create)
	+ [EPK](https://ipads.se.sjtu.edu.cn/_media/pub/members/2022_-_a_-_atc_-_epk.pdf)

### where the call stacks are in multithread programs
![s1](/assets/2023-11-10/s1.png)

- if we are to use pkey to protect the call stack of each threads, 
	+ how do we get the starting address of each stack
	+ how do we get the size of each stack
	+ after a new thread is created, how to protect the memory region 

### measure vhive's function invocation time

### chat with Tanvir
- a [paper](https://homes.cs.washington.edu/~arvind/papers/google-rpc.pdf) suggested from Tanvir
- Tanvir wanted to have a meeting for the 3 of us

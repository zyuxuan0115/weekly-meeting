---
layout: post
title:  "2023-10-6 setup nightcore"
date:   2023-10-6 1:53:46 -0500
categories: serverless functions
---

### Things to be discussed
- inject function's binary code
	+ source code (Sebastian)
		* [BOLT](https://github.com/facebookarchive/BOLT)
			- can move functions around, but the input must be a binary
			- because it needs the [ELF(Executable and Linkable Format)](https://en.wikipedia.org/wiki/Executable_and_Linkable_Format) from the binary
		* problem with nightcore
			- the serverless functions are [library code](https://github.com/ut-osa/nightcore/blob/asplos-release/examples/c/compile.sh)
			- [RPC](https://github.com/ut-osa/nightcore/blob/asplos-release/examples/c/foo.c#L39) -> normal function call 
				+ how to convert the arguments of RPC into the arguments of normal function call at binary level
		* other requirements
			- the code of system and serverless functions must be written in C/C++ (nightcore in C++)
			- in the docker container (or other containers), it must support `ptrace` function calls (Joe we can run `ptrace` in the container)
			- in the docker container (or other containers), BOLT must be able to run in the container. 
	+ performance (Joe) 
		* if we build this system, can we really improve performance?
			- Yes, we can.
				+ got the answer for [nightcore's paper](https://www.cs.utexas.edu/users/witchel/pubs/jia21asplos-nightcore.pdf)
	+ OS interrupt (Sebastian)
		* explain how code insertion work
		* MySQL: multithreads
	+ isolation (Sebastian)
		* After inserting function B to the address space of function A, B can see A's local variable stored on the stack. unsafe. 
		* thought 1: A creates a thread for B and the new thread runs B's code. Each thread has its own stack. 
			- But how to create a thread at binary level? 
		* thought 2: Shared memory.

### Things I've done
- change nightcore's script to make it only run `foo` and `bar` on different machines
- start from the simplest case
		* ![arch](/assets/2023-10-02/s2.png)
- will discuss with Haoran this afternoon

![s1](/assets/2023-10-12/s1.png)

![s2](/assets/2023-10-12/s2.png)


### Motivation about this research
- why do we need to merge serverless functions
	+ improve performance
		* inserting binary code of a function takes	3ms, according [RPG2 paper](https://www.overleaf.com/project/648b610e909cdabfc31502d1)
	

---
layout: post
title:  "2024-02-02 outline + other FaaS platform + MuCache"
date:   2024-01-29 1:53:46 -0500
categories: serverless functions
---

### Other C++ serverless platform
- [google functions framework C++](https://github.com/GoogleCloudPlatform/functions-framework-cpp)
- also, we don't need the FaaS runtime written in C/C++, we only need the serverless functions written in C/C++

### SoftBound paper
- [paper](https://llvm.org/pubs/2009-06-PLDI-SoftBound.pdf) / [github repo](https://github.com/santoshn/softboundcets-34)
- performance overhead is 67% for SPEC benchmark suite. 
	+ however, I don't think the overhead for serverless functions is as high as SPEC benchmarks.
	+ [SPEC benchmark suite](https://www.spec.org/cpu2017/Docs/overview.html#suites)
- works only with LLVM 3.4 (a very old version)

### Light-weight contexts
- [paper](https://www.usenix.org/system/files/conference/osdi16/osdi16-litton.pdf) / [webpage](https://www.cs.umd.edu/projects/lwc/)
	+ I like the idea of the paper but they only implemented lwc in FreeBSD.
		* you have to make sure your container's system is FreeBSD + lwc
- an example of the isolation of sensitive data
![s1](/assets/2024-01-29/s1.png)
- how do we use lwc to guarantee `isolation` when merging serverless func?
	+ assume we have 2 serverless functions A and B
	+ in `main`:
		* initialize serverless function A
		* `lwCreate` context for A
		* `if caller != -1 then run function call A`
		* erase the sensitive data in A
		* `lwRestrict` change the permission of the root lwc, so the root lwc won't have access to A's lwc.
		* then in the for loop
			- lwSwitch to A's lwc
			- lwSwitch to B's lwc
		* <strong>in the same way create function B's lwc</strong>
	+ on thing that need us to pay attention is
		* we need to use `lwRestrict` or `lwOverlay` to change the permission of memory region that stores the arguments we want to pass from A to B. 			 

### How do you get call graph of a distributed system 
- distributed tracing

### Thoughts about MuCache
- MuCache
	+ Wrapper
		* readset (per service)
			- request identifier : keys of data store within the request + call argument
		* per-request context 
			- the id of the request
				+ if 2 requests from the RPC in a service, will they have different id?
			- the hash value of the request’s arguments 
			- the caller of the request
			- visited services of the request and its subrequests
				+ what is a subrequest?
				+ how large could it be?
			- whether the current request is read-only	
	+ Cache Manager
- the diagram
![d1](/assets/2024-01-29/d1.png)
- things could be profiled and tuned
	+ ![d2](/assets/2024-01-29/d2.png)
	+ (3) use cache eviction rate to decide whether the current size of the cache is good enough
- a question about the call graph
	+ since the services are distributed on different machines, how do you get the centralized call graph? 

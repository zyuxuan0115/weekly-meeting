---
layout: post
title:  "2024-01-29 outline + other FaaS platform + mucache"
date:   2024-01-29 1:53:46 -0500
categories: serverless functions
---

### other C++ serverless platform
- [google functions framework C++](https://github.com/GoogleCloudPlatform/functions-framework-cpp)
- also, we don't need the FaaS runtime written in C/C++, we only need the serverless functions written in C/C++

### softBound paper
- [paper](https://llvm.org/pubs/2009-06-PLDI-SoftBound.pdf) / [github repo](https://github.com/santoshn/softboundcets-34)
- performance overhead is 67% for SPEC benchmark suite. 
	+ however, I don't think the overhead for serverless functions is as high as SPEC benchmarks.
	+ [SPEC benchmark suite](https://www.spec.org/cpu2017/Docs/overview.html#suites)
- works only with LLVM 2.4 (a very old version)

### light-weight contexts
- [paper](https://www.usenix.org/system/files/conference/osdi16/osdi16-litton.pdf)
	+ I like the idea of the paper but they only implemented lwc in FreeBSD.
		* you have to make sure your container's system is FreeBSD + lwc
	+ How to use it?
		* an example
		* ![s1](/assets/2024-01-29/s1.png)
		* just like Algorithm 3, in `main`:
			- initialize serverless function A
			- `lwCreate` context for A
			- `if caller != -1 then run function call A`
			- erase the sensitive data in A
			- `lwRestrict` change the permission of the root lwc, so the root lwc won't have access to A's lwc.
			- <strong>in the same way create function B's lwc</strong>
			 

### Do we have to be faster than nightcore?
- nightcore requires all containers to be running on the same machine
	+ nightcore never resolves the function cold start issue
	+ the function cold start issue is a larger issue compared with RPC overhead

### thoughts about mucache

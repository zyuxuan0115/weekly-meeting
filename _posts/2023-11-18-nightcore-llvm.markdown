---
layout: post
title:  "2023-11-18 nightcore & llvm"
date:   2023-11-18 1:53:46 -0500
categories: serverless functions
---
### more thoughts about pkey
- build a library just like libmpk
	+ a map maintains the relation between virtual key and physical key (1,2,3,...,15)
	+ `physical key -> virtual key`: distributes the 16 keys to a larger space.
		* harder for attackers to figure out the value of the virtual keys
	+ create a bunch of APIs that use the virtual key as the argument but have the same semantics as the original pkey calls (e.g., `pkey_alloc`, `pkey_mprotect`)
		* these APIs should be based on the original pkey calls. 
- have another `ld_preload library` that intercepts the calls of the original `pkey_set()`, `pkey_alloc()`, `pkey_mprotect()` etc. that directly executed by the user code.
	+ the technique is called [LD_PRELOAD trick](https://stackoverflow.com/questions/426230/what-is-the-ld-preload-trick)
	+ once a pkey call being executed from user code is detected, we disable it
	+ how do we distinguish whether a pkey call is from our new library code or from user code?
		* using `libunwind` to get the function caller info from the call stack.

### understand how nightcore's external call works

### measure vhive’s function invocation time

### compiler identifies the arguments of a RPC

### faster inter-thread synchronization 

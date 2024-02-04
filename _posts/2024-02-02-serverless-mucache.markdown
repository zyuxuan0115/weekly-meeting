---
layout: post
title:  "2024-02-02 FaaS + MuCache"
date:   2024-02-02 1:53:46 -0500
categories: serverless functions
---

### Other C++ serverless platform
We don't need the FaaS runtime written in C/C++, we only need the serverless functions written in C/C++
- [google functions framework C++](https://github.com/GoogleCloudPlatform/functions-framework-cpp)
	+ [this page](https://github.com/GoogleCloudPlatform/functions-framework-cpp/tree/main/examples/howto_use_legacy_code) shows an example about how to build C++ serverless functions using google's function-framework-cpp
- [openfaas](https://github.com/openfaas)
	+ In 2018 you create FaaS platform with openfaas in [this way](https://medium.com/@pavithra_38952/openfaas-on-docker-440541d635a2)
	+ deploy existing containers to OpenFaaS [webpage](https://www.openfaas.com/blog/porting-existing-containers-to-openfaas/)

### SoftBound paper
- [paper](https://llvm.org/pubs/2009-06-PLDI-SoftBound.pdf) / [github repo](https://github.com/santoshn/softboundcets-34)

### How do you get call graph of a distributed system 
- distributed tracing

### Light-weight contexts
- [paper](https://www.usenix.org/system/files/conference/osdi16/osdi16-litton.pdf) / [webpage](https://www.cs.umd.edu/projects/lwc/)
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


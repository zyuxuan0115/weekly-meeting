---
layout: post
title:  "2024-01-29 outline + other FaaS platform + mucache"
date:   2024-01-29 1:53:46 -0500
categories: serverless functions
---

### other C++ serverless platform
- [google functions framework C++](https://github.com/GoogleCloudPlatform/functions-framework-cpp)

### softBound paper
- [paper](https://llvm.org/pubs/2009-06-PLDI-SoftBound.pdf)
	+ performance overhead is 67% for SPEC benchmark suite. 
		* however, I don't think the overhead for serverless functions is as high as SPEC benchmarks.
	+ works only with LLVM 2.4 (a very old version)
- [github repo](https://github.com/santoshn/softboundcets-34)

### light-weight contexts
- [paper](https://www.usenix.org/system/files/conference/osdi16/osdi16-litton.pdf)

### thoughts about mucache

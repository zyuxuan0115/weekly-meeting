---
layout: post
title:  "2022-11-21 data cache"
date:   2022-11-20 13:13:46 -0500
categories: cont-opt
---
- Previous efforts on adding `__builtin_prefetch()` in the source code of `cc` and `pr` doesn't show significant performance improvement.
	* OoO will hide some of the memory latency. OoO only fails to hide latency when 
		+ (1) OoO is unable to unroll some loop, (2) there are some substantial computations
		+ use top-down to check whether these workloads are <em> memory latency bound </em> or <em> memory bandwidth bound </em>
		+ The orginal `gapbs/pr` workloads' top-down results for data cache
		![top-down pr omp](/assets/2022-11-21/pr_omp.png)
		+ The top-down results for `gapbs/pr` workloads' running in serial for the TOP 1 llc-load-miss location
		![top-down pr serial](/assets/2022-11-21/pr_serial.png)
	* wrong way to insert `__builtin_prefetch()`
		+ Leverage APT-GET or DMon's LLVM pass to decide the prefetch location. 
		+ APT-GET shows a significant performance improvement on `pr`, although they don't use the `gapbs/pr`
		+ APT-GET has a number of different inputs for `pr` and `bfs`. We can also tune the parameters of the inputs for other workloads.
		![apt-get inputs](/assets/2022-11-21/apt-get.png) 

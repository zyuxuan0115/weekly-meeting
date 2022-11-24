---
layout: post
title:  "2022-11-27 BOLT's dataAggregation"
date:   2022-11-22 12:53:46 -0500
categories: cont-opt
---
- In [DataAggregator::start()](https://github.com/zyuxuan0115/llvm-project/blob/main/bolt/lib/Profile/DataAggregator.cpp#L162)
  * `MainEventsPPI` can either be with Branch events or without Branch events. Normally it will contain Branch event, unless `-nl` is specified.  
	* 4 launchPerfProcess will be running. 
	* `MemEventsPPI` will not contain anything.

- In [DataAggregator::preprocessProfile](https://github.com/zyuxuan0115/llvm-project/blob/main/bolt/lib/Profile/DataAggregator.cpp#L468)
	* they have a lambda function [prepareToParse](https://github.com/zyuxuan0115/llvm-project/blob/main/bolt/lib/Profile/DataAggregator.cpp#L484).
  * Also [parse branch samples](https://github.com/zyuxuan0115/llvm-project/blob/main/bolt/lib/Profile/DataAggregator.cpp#L1395) 

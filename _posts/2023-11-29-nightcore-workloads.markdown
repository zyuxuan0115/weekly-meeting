---
layout: post
title:  "2023-11-24 nightcore-workloads"
date:   2023-11-24 1:53:46 -0500
categories: serverless functions
---

### From workload movieReviewing 
- I checked if the movieReview workload contains the invocation `invoke_func_fn` 
	+ My guess: `invoke_func_fn` is just the nightcore internal call

![a](/assets/2023-11-29/S1.png)
- only nightcore [FaasWorker.h](https://github.com/zyuxuan0115/nightcore-test/blob/main/moviereview_singlenode/DeathStarBench/mediaMicroservices/src/FaasWorker.h) contains [invoke_func_fn_](https://github.com/zyuxuan0115/nightcore-test/blob/main/moviereview_singlenode/DeathStarBench/mediaMicroservices/src/FaasWorker.h)

- what would the nightcore external call look like?

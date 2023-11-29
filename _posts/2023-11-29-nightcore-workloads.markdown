---
layout: post
title:  "2023-11-24 nightcore-workloads"
date:   2023-11-24 1:53:46 -0500
categories: serverless functions
---

### From workload movieReviewing 
- I checked if  
- only nightcore [FaasWorker.h](https://github.com/zyuxuan0115/nightcore-test/blob/main/moviereview_singlenode/DeathStarBench/mediaMicroservices/src/FaasWorker.h) contains [invoke_func_fn_](https://github.com/zyuxuan0115/nightcore-test/blob/main/moviereview_singlenode/DeathStarBench/mediaMicroservices/src/FaasWorker.h)

- My guess: `invoke_func_fn` is just the nightcore internal call

- what would the nightcore external call look like?

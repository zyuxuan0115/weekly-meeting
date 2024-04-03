---
layout: post
title:  "2024-04-03 DeathStarBench"
date:   2024-04-03 1:53:49 -0500
categories: serverless functions
---
### Beldi's serverless functions from DeathStarBench
- paper is [here](https://www.usenix.org/conference/osdi20/presentation/zhang-haoran)
- Beldi's modified DeathStarBench is [here](https://github.com/eniac/Beldi/tree/master/internal/media/core)
- Question: for example `MovieId` contains write (store the data to the database)
  + how do we handle these? 
	+ suggestion: allow data inconsistency.
- Question: which functions are going to be written in Rust?

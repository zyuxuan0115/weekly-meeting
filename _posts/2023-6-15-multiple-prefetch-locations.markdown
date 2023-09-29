---
layout: post
title:  "2023-6-15 multiple-prefetch-locations"
date:   2023-6-15 1:53:46 -0500
categories: data-cache
---

### prefetch for multiple locations

- sssp
	+ `./sssp 1 1 email-euAll 10`
	+ prefetch distance for both locations: 64

| | 0x4015fc | 0x401611 | 0x4015f8 |
|:---:|:---:|:---:|:---:|
| LLC misses/sec | 2055.194662 | 1444.749882 | 57.21327729 |

| | original | prefetch both | prefetch 0x4015fc | prefetch 0x401611 | 
|:---:|:---:|:---:|:---:|:---:|
| execution time <br> (second) | 20.945770 |  <strong>12.749177</strong> | 15.837548 | 19.714539 | 


- bc
  + `./bc 1 20000 500` 

| | 0x401561 | 0x401565 |
|:---:|:---:|:---:|
| LLC misses/sec | 2001.573352 | 1880.66186 | 
 
| | original | prefetch both |  prefetch 0x401561 | prefetch 0x401565 |
|:---:|:---:|:---:|:---:|:---:|
| execution time <br> (second) | 249.047044 | <strong>274.041806</strong> | 228.323513 |  238.846107 | 


### Change BOLT to support insert multiple locations
![multiple_vs](/assets/2023-06-15/inject_pref.png)

- If prefetch for multiple locations
	+ at runtime (PG^2), how to efficiently decide the optimal prefetch distances for these locations? 

- Question to Heiner & Saba
	+ Can APT-GET prefetch for multiple locations?
	+ if it cannot, we have a chance to outperform APT-GET
	![peaks](/assets/2023-06-15/peaks.png)



### randacc
![s1](/assets/2023-06-15/s1.png)

![randacc](/assets/2023-06-15/rand_part.png)

| | randacc | randacc.bolt | randaccswpf |
|:---:|:---:|:---:|:---:|
| 1 | 6.116 | 5.514 | 4.305 | 
| 2 | 6.676 | 5.555 | 3.371 | 
| 3 | 6.308 | 5.780 | 3.338 | 
| 4 | 6.719 | 5.589 | 4.320 | 
| 5 | 6.195 | 4.356 | 4.195 | 
| average | 6.4028 | 5.3588 | 3.9085 | 



### algorithm for inserting prefetch
- 2 kinds of workloads
	+ <strong>CRONO</strong>: indirect memory access
		* inject to outer loop 
		* <strong>pr, bc, bfs, sssp, dfs</strong>
			* need to test: can APT-GET ver. binary has performance improvement? 
	+ <strong>AinsworthCGO2017</strong>: indirect memory access & direct memory access
		* inject to inner loop
		* <strong>nas-is, nas-is, randacc, hashjoin2, graph-500</strong>

| | pagerank | bc | bfs | sssp | dfs | is | cg | randacc | hashjoin2 | graph-500 |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| prefetchable? | Y | Y | Y | Y | N | Y | Y | Y | Y | Y | 
| performance gain? | Y | Y | Y | Y | N/A | Y | N | Y | Y | N | 
| inner or outer loop?| O | O | O | O | N/A | I | I | I | I | I | 
| indirect or direct memory access?| I | I | I | I | I | D | D | I | D | ? | 
| multiple prefetch loc? | S | M | S | M | N/A | S | S | M | S | M | 
| notes |  | | performance gain only from synthetic inputs| | | | | | performance gain only in a few functions. No overall speedup| |



- 2 BOLT passes: 1 for inner loop, 1 for outer loop 
	+ should be mutual exclusive
		* because we either insert to outer loop, or insert to inner loop
		* we don't insert prefetch to both loops
	+ AinsworthCGO2017: insert to inner loop
		* nas-is, hashjoin2: only contain inner loop 
			* insert to inner loop
		* randacc, nas-cg, graph-500: nested loop
			* CRONO workloads also have nested loop, but we insert prefetch to outer loop
- <strong>Questions: how to decide when to insert to inner loop?</strong>
	+ always start with the Top-LLC-Miss location 
		* that's the only profile info we had at the very beginning
	+ if there is no outer loop, insert prefetch to the inner loop
	+ check the inner loop
		* from the Top-LLC-Miss load, we check the dependency of the instructions
		* until we find an instruction whose source registers are 
			* either loop induction variable
			* or loop invariant
			* randacc's case
			* ![randacc](/assets/2023-06-15/rand_part.png)
		* We stop there (at the sourse instruction) and check all source registers in the dependency chain
			* to see if there is any register is the loop induction variable for the outer loop
			* or is the dst reg of an instruction that only depends on the loop induction variable for the outer loop
			* sssp's example
			* ![sssp_1](/assets/2023-06-15/sssp_1.png)
			* bc's example
			* ![bc](/assets/2023-06-15/bc.png)
			* if there is 
				- check the other source registers in the dependency chain.
				- if all of them are loop invariant in the outer loop
				- add prefetch in the outer loop
				- otherwise add prefetch in the inner loop
				- ![cg](/assets/2023-06-15/cg_clang.png)
			* if there is no register that is the loop induction variable for the outer loop
				- add prefetch in the inner loop

| | pagerank | bc | bfs | sssp | dfs | is | cg | randacc | hashjoin2 | graph-500 |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| can APT-GET insert prefetch?| Y | Y | N | N | Y | Y | N | N | N | Y | 
| APT-GET has performance gain?| Y | N | N/A | N/A | Y | N | N/A | N/A | N/A | N | 



### How to measure APT-GET binary's performance?
- Produce an optimized binary for each input?
	+ impractical
	+ it takes APT-GET to spend more than 3 minutes (sometimes half of an hour for bc on syn inputs) to produce an optimized binary
		* most of the time was spending on analyzing the profile to compute an optimal prefetch distance
	+ in practical, if you produce an optimized binary each time before you used binary, your performance is actually
		* <strong>the time to produce the binary (>3 minutes) + the optimized execution time (45 seconds)</strong>
		* this is even longer than the original execution time (1 minute)!
	+ What I proposed:
		* run APT-GET on all inputs for just 1 time 
		* find the prefetch distance that occurs the most (mode) as the optimal prefetch distance
		* produce the optimized APT-GET binary
		* use the optimized APT-GET binary to measure the performance for all inputs
- In Joe's email to Heiner, he also wanted to generate an APT-GET binary for each input to measure the upper bound of pg^2's performance
	+ APT-GET will tell you different optimal prefetch distances on different runs even for the same input .
		* see 2022-12-12's blog
	+ solution:
		* run APT-GET with each input for at least 5 times 
		* find the optimal prefetch distance


### Better heuristic for determine prefetch distance
- 1 prefetch location
	* local optimal vs. global optimal?
	* ![localvsglobal](/assets/2023-06-15/localvsglobal.png)
	* in reality, there is such a case from `sssp + amazon0312` on cat16
	* ![amazon0312](/assets/2023-06-15/IMG_7510.png)
- 2 prefetch locations?	
	* how to decide the optimal prefetch distances in a faster way?





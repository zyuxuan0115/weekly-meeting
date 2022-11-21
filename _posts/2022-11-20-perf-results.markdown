---
layout: post
title:  "2022-11-21 continuous optimization"
date:   2022-11-20 13:13:46 -0500
categories: cont-opt
---
### diagram of continuous optimization

![diagram for cont-opt](/assets/2022-11-21/diagram-cont-opt.png)

#### 2 questions
- How to map profile1 to the original binary and then produce a BOLTed binary? 
	* Function Reordering: 
![function reordering](/assets/2022-11-21/func-reorder.png)
		+ Ocolos maintains a map table between the starting addr of original functions and the starting addr of BOLTed functions.
	* Basic Block Reordering:
![bb reordering](/assets/2022-11-21/bb-reorder.png)
		+ BOLT's BAT-dump maintains a map table between the branch insns in the original functions and branch insns in the BOLTed functions.
		+ BAT-dump related thing can be found [here](BAT-usage.html)
	* Use Ocolos and BAT-dump to change the records (addr recorded) in the profile data.
		+ Problem: the profile data recorded in the C1 round still use the old address of the original binary, even though some of the functions are moved by OCOLOS.
	* Use the changed profile data + orig binary => new BOLTed binary.

- How to performance code replacement (especially changing function pointers) on the C1 code layout?
	* In the OCOLOS paper, Joe has some thoughts in section 4.C.

### diagram of profile on C0 round and C1 round
![diagram for c0 c1 rounds](/assets/2022-11-21/c0-c1.png)


### 3 different profiles 
- profile for C0 
  * In the blue column, all addresses are lower than `0x3000000`.
  * ![perf script results with only events collected from mysqld](/assets/2022-11-21/c0.png)

- profile for C1
  * In the blue column, all addresses are still lower than `0x3000000`.
  * ![perf script results with only events collected from mysqld](/assets/2022-11-21/c1.png)

- profile for offline BOLTed mysqld
  * In the blue column, all addresses are greater than `0x4800000`.
  * ![perf script results with only events collected from mysqld](/assets/2022-11-21/bolt.png)


### perf2bolt on perf.data collected from C1 round
- perf2bolt on perf.data collected from C0 round
![perf2bolt on c0](/assets/2022-11-21/c0perf2bolt.png)

- perf2bolt on perf2.data collected from C1 round
![perf2bolt on c1](/assets/2022-11-21/c1perf2bolt.png)

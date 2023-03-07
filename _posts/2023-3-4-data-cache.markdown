---
layout: post
title:  "2023-3-4 CFG"
date:   2023-3-4 7:53:46 -0500
categories: data-cache 
---
### Some explaination of injecting prefetch to binary
- why small changes in the binary when prefetch distance changes?
	+ APT-GET injects prefetch instruction at a fixed location!
		* we've checked the code of APT-GET pass, the "prefetch function" is always injected before the `curLoad` instruction
	+ APT-GET only prefetches `a[b[i]]` in for loops!
	+ the address to be prefetched can be represented in this way: address = `f(d, i)` 
		* where `d` is the prefetch distance suggested by APT-GET
		* `i` is the loop variable
	+ we can further conclude that the address to be prefetched = `c1 + c2 * d + g(i)`
		* where `c1` and `c2` are constant
		* `d` is the prefetch distance
		* `i` is the loop variable
	+ to compute `g(i)`, APT-GET needs to inject some arithmetic instructions in the for loop.
		* sometimes to compute `g(i)`, APT-GET may also change the shape of part of the CFG (compared with the original binary) 
			- I remember in the APT-GET's code it has `createSwitch` and `createICMP` that may change the shape of the CFG
	+ the compiler will turn `c1 + c2 * d` into a constant variable
- more things about `g(i)` and `d`
	+ the change of `g(i)` is fixed (compared with the original binary), because it does not change with the prefetch distance `d`
		* this means the instructions added by APT-GET and CFG reshaped by APT-GET will not change no matter what value of `d` is
	+ so when we change the binary, we only need to change the constant value `c1 + c2 * d`, namely `0x14f0`in the below assembly code 
```
mov 0x14f0(%r12,%rdx,8),%rcx
prefetchT0 (%rcx)
```

### Another "intuition" of code generation
- basically it's impossible to make the optimized binary close to the original binary.
	+ because when APT-GET generates code to compute `g(i)`, it will always add new instructions and sometimes reshape part of the CFG.
	+ even if the compiler applies almost the same register allocation, there are still too much difference. (This may not be true, just my intuition)
- so I'd like to think this problem reversely.
	+ <strong>if we cannot make the optimized binary close to the original, how about making the original close to the optimized? </strong> (according to our previous analysis, this path is much easier!!)

### test the different workloads' optimized binaries
- pagerank (prefetch distance=32 vs 100) 
![datacache1](/assets/2023-03-04/pr.png)
- bc (prefetch distance=32 vs 100) 
![datacache2](/assets/2023-03-04/bc.png)
- dfs (prefetch distance=20 vs 100)
![datacache3](/assets/2023-03-04/dfs.png)

### General case (other types of data-cache prefetch)
- according to the 5th page of the [DMon paper](https://web.eecs.umich.edu/~takh/papers/khan-dmon-osdi-2021.pdf), the 
![s1](/assets/2023-03-04/s1.png)
![s2](/assets/2023-03-04/s2.png)

hexdump 
objdump -d -F
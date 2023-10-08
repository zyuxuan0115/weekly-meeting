---
layout: post
title:  "2023-10-2 nightcore performance"
date:   2023-10-2 1:53:46 -0500
categories: serverless functions
---
### nightcore related
- [nightcore paper](https://www.cs.utexas.edu/users/witchel/pubs/jia21asplos-nightcore.pdf)
- [nightcore github repo](https://github.com/ut-osa/nightcore)
	+ the whole nightcore is written in C++!
- [nightcore benchmark github repo](https://github.com/ut-osa/nightcore-benchmarks) 

### RPC overhead from nightcore
![RPC-overhead](/assets/2023-10-02/s1.png)

### how to deploy nightcore
- I studied the script of nightcore-benchmark to see how to 
	+ setup a cluster
	+ deploy nightcore on a cluster
	+ run service on the deployed nightcore system
	+ ![arch](/assets/2023-10-02/s2.png)

### RPC graph
- ![RPC graph](/assets/2023-10-02/s3.png)
- what we want to do
	+ change [this RPC](https://github.com/ut-osa/nightcore/blob/asplos-release/examples/c/foo.c#L39) to be a normal function call in Foo's address space

```c++
    int ret = context->invoke_func_fn(
        context->caller_context, "Bar", input, input_length,
        &bar_output, &bar_output_length);
```

### questions about combining nightcore with our system
- `Foo` and `Bar` are compiled into libraries.
	+ if we want to insert `Bar` into `Foo`'s address space
		* the target address of the branch in `Bar`
		* PC-relative instructions need to be updated
- How to detect `Foo`'s RPC and change it into a call instruction?
	+ need to detect and erase the RPC that calls `Bar`
	+ need to copy the arguments of the RPC to the normal call instructions
	+ if RPC replaced with normal call, the code after RPC might also have their PC updated
		- PC relative instructions needs to be updated again.
- Can BOLT change the code layout of libraries?
- Is updating the text section of a program running in a docker container feasible?
	+ previously we only update programs that are running without containers. 

### things about building a RPC graph
- The RPC graph must be updated at some point after the deployment.
	+ so there is a need for us to rebuid a new call graph and move serverless functions around
- The RPC graph cannot be updated too frequently. 
	+ because injecting functions to the address space of other functions have overhead
	+ the overhead of inserting functions is much higher than a RPC

	

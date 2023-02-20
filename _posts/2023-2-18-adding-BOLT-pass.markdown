---
layout: post
title:  "2023-2-18 adding new pass to BOLT"
date:   2023-2-18 7:53:46 -0500
categories: data-cache 
---
### APT-GET's Prefetch
- What is a prefetch? 
	+ APT-GET's prefetch injection [code](https://github.com/upenn-acg/floar/blob/master/apt-get/SWPrefetchingLLVMPass/SWPrefetchingLLVMPass.cpp#L388)
		* in the [code](https://github.com/upenn-acg/floar/blob/master/apt-get/SWPrefetchingLLVMPass/SWPrefetchingLLVMPass.cpp#L579), prefetch is a function
			+ can we inject a function call at binary code level?
			+ do we also need to inject the `prefetch()` function itself into the text section? or is this function in the library?
			+ and then create a call instruction that points to `prefetch()`?
			+ except for `prefetch()` function, is there any other instructions we need to insert?
			+ last time from the `objdump`'s assembly code, I remember `prefetch` is an instruction???
		* [injectPrefetch](https://github.com/upenn-acg/floar/blob/master/apt-get/SWPrefetchingLLVMPass/SWPrefetchingLLVMPass.cpp#L639) function is a recursive call
			+ what information do we expect `APT-GET` to provide us?
			+ the location for injecting the prefetch function? 
				- what can be used to represent the location?
				- Basic Block + offset?
				- a specific instruction? 

### Other paper also leverages BOLT 
![screenshot](/assets/2023-02-18/screenshot-1.png)
![screenshot](/assets/2023-02-18/screenshot-2.png)
- Do we also want to add a pass or directly modify BOLT's existing code?

### llvm's MCInst
- Reference for llvm's MCInst is [here](https://llvm.org/doxygen/classllvm_1_1MCInst.html)
- Reference for `X86InsertPrefetch` pass is [here](https://llvm.org/doxygen/X86InsertPrefetch_8cpp_source.html)
- Search `MCInst` and `New` keyword in all BOLT's source code.
	+ I had the following results:

```cpp
lib/Core/BinaryBasicBlock.cpp:  MCInst NewInst;
lib/Passes/Instrumentation.cpp:  for (MCInst &NewInst : Instrs) {
lib/Passes/Instrumentation.cpp:      MCInst NewInst;
lib/Passes/ShrinkWrapping.cpp:    MCInst NewInst;
```

```cpp
void BinaryBasicBlock::addBranchInstruction(const BinaryBasicBlock *Successor) {
  assert(isSuccessor(Successor));
  BinaryContext &BC = Function->getBinaryContext();
  MCInst NewInst;
  std::unique_lock<llvm::sys::RWMutex> Lock(BC.CtxMutex);
  BC.MIB->createUncondBranch(NewInst, Successor->getLabel(), BC.Ctx.get());
  Instructions.emplace_back(std::move(NewInst));
}

void BinaryBasicBlock::addTailCallInstruction(const MCSymbol *Target) {
  BinaryContext &BC = Function->getBinaryContext();
  MCInst NewInst;
  BC.MIB->createTailCall(NewInst, Target, BC.Ctx.get());
  Instructions.emplace_back(std::move(NewInst));
}
```

- Then I looked into the [BinaryBasicBlock.cpp](https://github.com/zyuxuan0115/llvm-project-pg-square/blob/main/bolt/lib/Core/BinaryBasicBlock.cpp) code
	+ In the code 
		* they first declare new `MCInst`
		* then create this instruction as a TailCall `BC.MIB->createTailCall(NewInst, Target, BC.Ctx.get());`
	+ seems like <strong>BC.MIB</strong> is the place we should pay attention to
		* are there other types of `MCInst` that we can create?
		* so I further checked [BC.MIB](https://github.com/zyuxuan0115/llvm-project-pg-square/blob/main/bolt/include/bolt/Core/BinaryContext.h#L587), and find
			- std::unique_ptr<MCPlusBuilder> MIB;
	+ so it seems that we should look into the `MCPlusBuilder` class
		* I looked into [MCPlusBuilder.h](https://github.com/zyuxuan0115/llvm-project-pg-square/blob/main/bolt/include/bolt/Core/MCPlusBuilder.h), and there are
			- [createNoop](https://github.com/zyuxuan0115/llvm-project-pg-square/blob/main/bolt/include/bolt/Core/MCPlusBuilder.h#L1440) function
				* is injecting a `NOOP` enough?
			- [createCall](https://github.com/zyuxuan0115/llvm-project-pg-square/blob/main/bolt/include/bolt/Core/MCPlusBuilder.h#L1474) function
				* if we want to inject a `prefetch` function call. The `MCSymbol *Target` (a.k.a `prefetch`) is not in the symbol table?
				* to add arguments to this call, we may need other instructions? (e.g. `createLoad` to load data (argument) into the corresponding register?)
				* the arguments of the call need to be passed from `APT-GET`'s [code](https://github.com/upenn-acg/floar/blob/master/apt-get/SWPrefetchingLLVMPass/SWPrefetchingLLVMPass.cpp#L589)
				* what if an argument of `prefetch` should be put into a specific register, but that register is currently in use?
			- however, in `APT-GET` they also inject other instructions:
				* [CreateInBoundsGEP](https://github.com/upenn-acg/floar/blob/master/apt-get/SWPrefetchingLLVMPass/SWPrefetchingLLVMPass.cpp#L525), seems like something related to `phi node`
				* [CreateAdd](https://github.com/upenn-acg/floar/blob/master/apt-get/SWPrefetchingLLVMPass/SWPrefetchingLLVMPass.cpp#L461)
				* [CreateICmp](https://github.com/upenn-acg/floar/blob/master/apt-get/SWPrefetchingLLVMPass/SWPrefetchingLLVMPass.cpp#L466)
				* Why does `APT-GET` also inject other instructions? Do we also need to inject these instructions?
		* Both `createNoop` and `createCall` are virtual functions 
			- So I found functions that override these virtual functions
			- they are [here](https://github.com/zyuxuan0115/llvm-project-pg-square/blob/main/bolt/lib/Target/X86/X86MCPlusBuilder.cpp#L2915) and [here](https://github.com/zyuxuan0115/llvm-project-pg-square/blob/main/bolt/lib/Target/X86/X86MCPlusBuilder.cpp#L2522) 

```cpp
bool createNoop(MCInst &Inst) const override {
    Inst.setOpcode(X86::NOOP);
    return true;
}
```

- I also looked into [BinaryBasicBlock.h](https://github.com/zyuxuan0115/llvm-project-pg-square/blob/main/bolt/include/bolt/Core/BinaryBasicBlock.h#L766)
	+ it also has `insertInstruction()` to insert a new `MCInst`. 
```cpp
iterator insertInstruction(iterator At, MCInst &NewInst) {
	adjustNumPseudos(NewInst, 1);
	return Instructions.insert(At, NewInst);
}
```

### Questions didn't get answered on our slack channel
- If we want to inject prefetch at runtime, it means the workloads should be always running and can accept any inputs. But in reality, the workloads we run (`pr`, `bfs`, `bc`) are not always running services.
	+ in practice, is there any case that requires such workloads to be running forever?
	+ If there is no case that requires pr to be an always running service, 
		* Why can't people produce different optimized binaries for different inputs statically and then use them  
		* (for `pr`’s case, we actually only need 2 optimized binaries since there are 2 optimal prefetch ranges; for other workloads, we only need an optimized binary and an original binary). 
		* So why do we want to inject prefetch at runtime? We can just change the binary when we run `pr` , `bfs` with different inputs.
	+ if in reality no people use these workloads as always running service, I think the only possible case for us to apply dynamic optimization is 
		+ there is <strong>a workload + large enough input running for a very long time</strong>. 
		+ Before the execution, we don’t know whether prefetch can benefit the performance or be harmful to the performance. 
		+ So we use performance counters to decide whether we want to inject prefetch insn/eliminate the previously injected prefetch insn, and then we inject/eliminate prefetch insn at runtime. 


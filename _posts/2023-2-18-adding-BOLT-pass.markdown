---
layout: post
title:  "2023-2-18 adding new pass to BOLT"
date:   2023-2-18 7:53:46 -0500
categories: data-cache 
---
### How to add a new pass to BOLT
![screenshot](/assets/2023-02-18/screenshot-1.png)
![screenshot](/assets/2023-02-18/screenshot-2.png)

### llvm's MCInst
Reference for llvm's MCInst is [here](https://llvm.org/doxygen/classllvm_1_1MCInst.html)
- Search `MCInst` and `New` keyword in all BOLT's source code.
	+ I had the following results:
```
lib/Core/BinaryBasicBlock.cpp:  MCInst NewInst;
lib/Passes/Instrumentation.cpp:  for (MCInst &NewInst : Instrs) {
lib/Passes/Instrumentation.cpp:      MCInst NewInst;
lib/Passes/ShrinkWrapping.cpp:    MCInst NewInst;
```


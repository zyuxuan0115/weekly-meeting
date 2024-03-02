---
layout: post
title:  "2024-03-01 DeathStarBench"
date:   2024-03-01 1:53:46 -0500
categories: serverless functions
---


## C + SoftBound + Rust
- works well

### concern about merging C and Rust functions
- How do they handle strings since we need to transfter strings from caller to callee
- this is Rust's IR code of a string 

```llvm
@alloc1 = private unnamed_addr constant <{ [22 x i8] }> <{ [22 x i8] c"Hello, I'm rust code!\0A" }>, align 1
```

- this is C's IR code of a string

```llvm
@.str = private unnamed_addr constant [20 x i8] c"Hello, I'm C code!\0A\00", align 1
```

## DeathStarBench

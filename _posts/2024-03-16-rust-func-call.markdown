---
layout: post
title:  "2024-03-16 RPC in rust"
date:   2024-03-16 1:53:49 -0500
categories: serverless functions
---
### Rust
#### libcurl for Rust
- [Easy](https://docs.rs/curl/latest/curl/easy/struct.Easy.html)
- [Easy2 trait Handler](https://docs.rs/curl/latest/curl/easy/trait.Handler.html)
- using libcurl to build RPC for serverless function in Rust
	+ done. code is [here]()

#### how to use rustc to compile files from other crate
- [stackoverflow](https://stackoverflow.com/questions/61987049/why-building-with-rustc-command-cannot-see-crates)

A concrete example of compiling a single library and a single executable using that library:

```rust
> rustc --edition=2018 --crate-type=rlib --crate-name library_example src/lib.rs -o libmy_library.rlib
> rustc --edition=2018 --extern library_example=libmy_library.rlib examples/main.rs
```

- [How to compile Rust to LLVM bitcode including dependencies?](https://stackoverflow.com/questions/69042049/how-to-compile-rust-to-llvm-bitcode-including-dependencies)

```bash
> RUSTFLAGS="--emit=llvm-ir" cargo build
> llvm-link target/release/deps/*.ll > withdeps.ll
```

#### Merge 2 Rust serverless functions
![d1](/assets/2024-03-16/d1.png)
- [the difference between CallInst and InvokeInst](https://stackoverflow.com/questions/35368366/call-vs-invoke-in-ir-codes-of-llvm)

#### the version number conflict
rustc version 1.46 is based on LLVM 10 (for SoftBound)
rustc version 1.76 (the current version) is based on LLVM 17 (for libcurl - to construct the RPC)

### LLVM
#### LLVM17 & new LLVM pass manager
- [llvm17's webpage](https://github.com/llvm/llvm-project/releases/tag/llvmorg-17.0.5)
- [new llvm pass tutorial](https://llvm.org/docs/WritingAnLLVMNewPMPass.html)

#### How to add LLVM Instrincs
- [llvm.memcpy intrincs](https://llvm.org/docs/LangRef.html#id2102)
- [stackoverflow example](https://stackoverflow.com/questions/11985247/llvm-insert-intrinsic-function-cos)
- [sample code to create IR builder](https://gist.github.com/seven1m/2ca74265cca9ef6f493ef1de87e9252d#file-hello_world_llvm-cpp-L14)

#### Difference between CallInst & InvokeInst
- [Call vs Invoke in IR codes of LLVM](https://stackoverflow.com/questions/35368366/call-vs-invoke-in-ir-codes-of-llvm)
- If invoke is done, it will jump to another basic block based on whether the invoke is successful or not.
- `InvokeInst` must be a terminator of a basic block

### Link 2 Rust function
- [manually linking rust binaries to support out of tree llvm passes](https://medium.com/@squanderingtime/manually-linking-rust-binaries-to-support-out-of-tree-llvm-passes-8776b1d037a4)

#### GDB tricks 
[use gdb's GUI](https://stackoverflow.com/questions/53167123/assembly-gdb-switch-between-gui-tables)
- `ctrl+x`, then `2`
- `layout regs`
- `focus asm`


---
layout: post
title:  "2024-03-16 RPC in rust"
date:   2024-03-16 1:53:49 -0500
categories: serverless functions
---

### libcurl for Rust
- [Easy](https://docs.rs/curl/latest/curl/easy/struct.Easy.html)
- [Easy2 trait Handler](https://docs.rs/curl/latest/curl/easy/trait.Handler.html)
- using libcurl to build RPC for serverless function in Rust
	+ done. code is [here]()

### how to use rustc to compile files from other crate
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

### LLVM17 & new LLVM pass manager
- [llvm17's webpage](https://github.com/llvm/llvm-project/releases/tag/llvmorg-17.0.5)
- [new llvm pass tutorial](https://llvm.org/docs/WritingAnLLVMNewPMPass.html)

---
layout: post
title:  "2024-03-01 make function calls another function"
date:   2024-03-01 1:53:46 -0500
categories: serverless functions
---
## Make C function call another serverless function 
### libcurl
- Using [libcurl](https://curl.se/libcurl/) can easily send http get request in C
	+ the example of using callback function is [here](https://curl.se/libcurl/c/ftpget.html)
- if a function is invoked within kubernetes cluster, to call the function, the REST api is:
  + `http://gateway.openfaas.svc.cluster.local.:8080`
  + reference is [here](https://docs.openfaas.com/reference/rest-api/#:~:text=Functions%20can%20be%20invoked%20by,path%20to%20the%20gateway%2    0URL.&text=If%20no%20namespace%20is%20specified,%2Fasync%2Dfunction%2FNAME.)
- test SoftBound on C function with `libcurl`
  + unfortunately, softbound doesn't support callback functions, and libcurl needs callback function
    * solution 1: not to use `libcurl` - doable but painful
    * solution 2: make RPC call a standalone static library and after softbound's pass is done, link that static library.  
      - because the RPC interface is developed by us, so we can guarantee it is always safe.

- the update of function from `faas-cli` command is very slow.
	+ can we update the function by logging into kubernetes and update local docker directly?

## Make Rust function call another serverless function
### From Rust [tutorial](https://doc.rust-lang.org/book/)
#### Chapter 2
- if not specify, everything is inmutable
- an associated function is a function that’s implemented on a type: `String::new`
- `enum` is a type that can be in one of multiple possible states.
  + Result’s variants are `Ok` and `Err`. The `Ok` variant indicates the operation was successful, and inside `Ok` is the successfully generated value. The `Err` variant means the operation failed, and `Err` contains information about how or why the operation failed.
  + expect will cause the program to crash and display the message that you passed as an argument to expect
- running the `cargo doc --open` command will build documentation provided by all your dependencies locally and open it in your browser
- `let guess: u32 = guess.trim().parse().expect("Please type a number!");`
  + convert a rust string to an integer

#### Chapter 3
- You aren’t allowed to use `mut` with constants, and the type of the value must be annotated.
- By using `let`, we can perform a few transformations on a value but have the variable be immutable after those transformations have been completed.
- declare an array type

```rust
let a: [i32; 5] = [1, 2, 3, 4, 5];
let a = [3; 5]; // a = [3,3,3,3,3];
```

- In function signatures, you must declare the type of each parameter.
- If you add a semicolon to the end of an expression, you turn it into a statement, and it will then not return a value.

### libcurl for rust
- [The link](https://docs.rs/curl/latest/curl/)
- [example](https://crates.io/crates/curl)

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


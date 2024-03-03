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
  + http://gateway.openfaas.svc.cluster.local.:8080
	+ reference is [here](https://docs.openfaas.com/reference/rest-api/#:~:text=Functions%20can%20be%20invoked%20by,path%20to%20the%20gateway%2    0URL.&text=If%20no%20namespace%20is%20specified,%2Fasync%2Dfunction%2FNAME.)

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


---
layout: post
title:  "2024-03-01 make function calls another function"
date:   2024-03-01 1:53:46 -0500
categories: serverless functions
---
## OpenFaaS
### Make OpenFaaS work on remote cluster (multiple machines)
- previously [KinD](https://kind.sigs.k8s.io/) can only deploy local kubernetes cluster
- using [k3sup](https://github.com/alexellis/k3sup) to build kubernetes cluster
	+ successful (now we can have multiple machines join the cluster)
- deploy OpenFaaS to the kubernetes cluster 


### can we directly change docker image?
- Yes we can. 2 options:
  + make kubernetes update the docker image
    * Reference (finally proved to be useless)
      - make kubernetes to update the image: [stackoverflow](https://stackoverflow.com/questions/40366192/kubernetes-how-to-make-deployment-to-update-image)
      - [The kubernetes documentation about how to use it to update the image](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#-em-restart-em-)
  + make OpenFaaS update the docker image (see the following example)

#### an example of update functions 
- actually it's just an update of the docker image
- this example: just change the output string
- finally we want unmerged function's docker image -> merged function's docker image

```bash
curl -i \
-X PUT \
http://admin:$PASSWORD@127.0.0.1:8080/system/functions \
-H 'Content-Type: application/json' \
-d '{"service":"hello-c","image":"zyuxuan0115/hello-c-new:latest","fprocess":"main","labels":{},"annotations":{}}'
```

## LLVM 

![d2](/assets/2024-03-06/d2.png)

### test C + SoftBound + Rust (simle example)
- works well

![d1](/assets/2024-03-06/d1.png)

### make C serverless function call another C serverless function 
#### libcurl
- Using [libcurl](https://curl.se/libcurl/) can easily send http get request in C
	+ the example of using callback function is [here](https://curl.se/libcurl/c/ftpget.html)
- if a function is invoked within kubernetes cluster, to call the function, the REST api is:
  + `http://gateway.openfaas.svc.cluster.local.:8080`
  + reference is [here](https://docs.openfaas.com/reference/rest-api/#:~:text=Functions%20can%20be%20invoked%20by,path%20to%20the%20gateway%20URL.&text=If%20no%20namespace%20is%20specified,%2Fasync%2Dfunction%2FNAME.)

#### test SoftBound on C function with `libcurl`
- unfortunately, softbound doesn't support callback functions, and libcurl needs callback function
  + solution 1: not to use `libcurl` - doable but painful
  + solution 2: make RPC call a standalone static library and after softbound's pass is done, link that static library.  
    * because the RPC interface is developed by us, so we can guarantee it is always safe.

#### C code before merge

```c
// caller
int main(){
  // -- do stuff

  char input = "I'm caller\n";
  char* output;
  // call another function
  // assume callee function's URL is 
  // http://IP:8080/function/callee
  make_RPC("callee", &input, &output);

  // -- do stuff based on `output` 
  return 0;
}
```

```c
// callee
int main(){
  // get input string from caller via STDIN
  // OpenFaaS runtime will hanle the conversion of 
  // message from caller's input arg to STDIN
  // so here input should be "I'm caller\n" 
  char input[1024];
  read(STDIN, input);

  // -- do stuff based on `input`

  // send the output back to caller via STDOUT
  // OpenFaaS runtime will hanle the conversion of 
  // function's STDOUT to message that will be send back 
  char* output = (char*)malloc(size);
  strcpy(output, ...);
  printf("%s", output);
  
  return 0;
}
```

#### C code after merge

```c
// caller
int main(){
  // -- do stuff

  char input = "I'm caller\n";
  char* output;
  // call another function
  callee(&input, &output);

  // -- do stuff based on `output` 
  return 0;
}

// callee
int callee(char* input, char** output){
  // -- do stuff based on `input`

  // write to output
  char* buffer = (char*)malloc(size);
  strcpy(buffer, ...);
  *output = buffer;
  return 0;
}
```


### make Rust function call another Rust serverless function
#### libcurl for rust
- [The link](https://docs.rs/curl/latest/curl/)
- [example](https://crates.io/crates/curl)

### concern about merging C and Rust functions
- How do they handle strings since we need to transfter strings from caller to callee
- this is C code 

```c
int main(void){
  char* message = (char*) malloc(sizeof(char)*1200);
  memset(message, 0, 1200);
  strcpy(message, "I'm caller-c function: ");
  char* output;
  make_RPC("callee-c", message, &output);
  printf("%s", output);
  return 0;
}
```

```llvm
%18 = call i32 @make_RPC(
  i8* getelementptr inbounds ([9 x i8], [9 x i8]* @.str.3, i64 0, i64 0), 
  i8* %17, 
  i8** %5)
```

- this is Rust's code and IR

```rust
fn add_prefix(orig_str: &str) -> String {
  let mut new_string = String::from("You just said: ");
  new_string.push_str(orig_str);
  new_string
}

fn main() {
    let orig_string = String::from("Hello, world!");
    let new_string = add_prefix(&orig_string);
    println!("{}", new_string);
}
```

```llvm
invoke void @_ZN4main10add_prefix17h151f354e2fc0a8a2E(
  %"alloc::string::String"* noalias nocapture sret dereferenceable(24) %new_string, 
  [0 x i8]* noalias nonnull readonly align 1 %_4.0, 
  i64 %_4.1)
```

- however any rust type can be converted into a combination of c types with metadata
	+ rust types in LLVM IR is just a alias of a combination of C types

```llvm
%"alloc::vec::Vec<u8>" = type { [0 x i64], { i8*, i64 }, [0 x i64], i64, [0 x i64] }
%"alloc::string::String" = type { [0 x i64], %"alloc::vec::Vec<u8>", [0 x i64] }
```

- references: 
  + [add metadata: stackoverflow](https://stackoverflow.com/questions/13425794/adding-metadata-to-instructions-in-llvm-ir)
  + [metadata: LLVM](https://llvm.org/docs/LangRef.html#metadata)
  + [Rust: How do I convert a &cstr into a String and back with ffi?](https://stackoverflow.com/questions/24145823/rust-how-do-i-convert-a-cstr-into-a-string-and-back-with-ffi)

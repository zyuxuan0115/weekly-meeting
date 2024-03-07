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

### An example of update functions 
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

## Make C serverless function call another serverless function 
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

### C code before merge

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

### C code after merge

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


## Make Rust function call another serverless function
### libcurl for rust
- [The link](https://docs.rs/curl/latest/curl/)
- [example](https://crates.io/crates/curl)

## C + SoftBound + Rust
- works well

![d1](/assets/2024-03-06/d1.png)

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

## Can we directly change docker image
- Yes we can.

### Reference (finally proved to be useless)
- make kubernetes to update the image: [stackoverflow](https://stackoverflow.com/questions/40366192/kubernetes-how-to-make-deployment-to-update-image)
- [The kubernetes documentation about how to use it to update the image](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#-em-restart-em-)

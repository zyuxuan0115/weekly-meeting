---
layout: post
title:  "2024-03-24 merge Rust and C"
date:   2024-03-24 1:53:49 -0500
categories: serverless functions
---
### Rust caller calls C callee
#### Caller pseudo code

```rust
// original caller
fn main(){
    let input: String = {some expression};
    let output: String = make_rpc("callee", input);
}

// modified caller
fn main(){
    let input: String = {some expression};
    let output: String = callee(input);
}
```

#### Callee pseudo code

```rust
// original callee
void main(){
    char* input = get_arg_from_caller();
    ...
    char* output = {some expression};
    send_return_value_to_caller(output);
}

// modified callee
char* callee_c(char* input){
    ...
    char* output = {some expression};
    return output;    
}

fn callee(input_rust: String) -> String {
    let input_cstring: CString = CString::new(&input[..]).unwrap();
    let input: *const c_char = input_cstring.as_ptr();

    // this call of callee_c is in IR so we don't need 
    // FFI to convert it to a C function
    let output: *const c_char = callee_c(input);

    let return_value: String = CStr::from_ptr(output).to_str().unwrap().to_owned();
    return_value
}
```

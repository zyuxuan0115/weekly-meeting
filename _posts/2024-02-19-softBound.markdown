---
layout: post
title:  "2024-02-19 SoftBound"
date:   2024-02-19 1:53:46 -0500
categories: serverless functions
---

### Other C++ serverless platform
We don't need the FaaS runtime written in C/C++, we only need the serverless functions written in C/C++
- [google functions framework C++](https://github.com/GoogleCloudPlatform/functions-framework-cpp)
	+ [this page](https://github.com/GoogleCloudPlatform/functions-framework-cpp/tree/main/examples/howto_use_legacy_code) shows an example about how to build C++ serverless functions using google's function-framework-cpp
	+ <strong>Problem</strong>: need to deploy your serverless functions on their platform
		* [build docker image and push to google's repo](https://cloud.google.com/artifact-registry/docs/docker/pushing-and-pulling)
		* [Configure remote repository authentication to Docker Hub](https://cloud.google.com/artifact-registry/docs/repositories/configure-remote-auth-dockerhub)
	+ [how to install google cloud cli](https://stackoverflow.com/questions/23247943/trouble-installing-google-cloud-sdk-in-ubuntu)

```bash
> curl https://dl.google.com/dl/cloudsdk/release/install_google_cloud_sdk.bash | bash
> source ~/.bashrc
> gcloud auth login
```

- [openfaas](https://github.com/openfaas): there is no tutorial about how to run openfaas at all
	+ [install OpenFaas](https://gcore.com/learning/create-serverless-functions-with-openfaas/)
	+ In 2018 you create FaaS platform with openfaas in [this way](https://medium.com/@pavithra_38952/openfaas-on-docker-440541d635a2)
	+ deploy existing containers to OpenFaaS [webpage](https://www.openfaas.com/blog/porting-existing-containers-to-openfaas/)
	+ [docker support](https://docs.openfaas.com/languages/dockerfile/)
	+ [older version](https://www.digitalocean.com/community/tutorials/how-to-install-and-secure-openfaas-using-docker-swarm-on-ubuntu-16-04)
- [ironFunction](https://github.com/iron-io)
	+ [run IronFunction as a scheduler on top of Docker Standalone Swarm cluster](https://github.com/iron-io/functions/tree/master/docs/operating/docker-swarm)
    * [scheduling service on a docker swarm mode cluster](https://semaphoreci.com/community/tutorials/scheduling-services-on-a-docker-swarm-mode-cluster)
	+ [Protect the Docker daemon socket](https://docs.docker.com/engine/security/protect-access/)
		* that's why we need to set `DOCKER_TLS_VERIFY`, `DOCKER_HOST`, `DOCKER_CERT_PATH` etc. 
    * also there is a [stackoverflow post for this](https://stackoverflow.com/questions/38286564/docker-tls-verify-docker-host-and-docker-cert-path-on-ubuntu)
	+ the nice thing of `ironFunction`
		* it convert the user request into `STDIN`, and the output to user only need to be set to `STDOUT`

### SoftBound paper
- [paper](https://llvm.org/pubs/2009-06-PLDI-SoftBound.pdf) / [github repo](https://github.com/santoshn/softboundcets-34)
- On [this line](https://github.com/santoshn/softboundcets-34/blob/master/softboundcets-llvm-clang34/tools/softboundcets/main.cpp), it shows how different llvm passes are added.

```c++
const std::string &ModuleDataLayout = M1.get()->getDataLayout();
TD = new DataLayout(ModuleDataLayout);
Passes.add(TD);

TargetLibraryInfo *TLI = new TargetLibraryInfo(Triple(M1.get()->getTargetTriple()));
Passes.add(TLI);
Passes.add(new InitializeSoftBoundCETS());
Passes.add(new SoftBoundCETSPass());

if(fix_byval_attributes){
  Passes.add(new FixByValAttributesPass());
}

Passes.add(createBitcodeWriterPass(Out->os()));
Passes.run(*M1.get());

```

SoftBound
 
```bash
clang -fsoftboundcets test.c -o test -L<git_repo>/softboundcets-lib -lm -lrt
```

- SoftBound doesn't take the generated IR as the input.
	+ this is due to the fact that version of LLVM is too old 
	+ but they still have the code for the pass of softbound
		* it's at [here](https://github.com/santoshn/softboundcets-34/tree/master/softboundcets-llvm-clang34/lib/Transforms/SoftBoundCETS) and [here](https://github.com/santoshn/softboundcets-34/tree/master/softboundcets-llvm-clang34/include/llvm/Transforms/SoftBoundCETS)
		* the other of the passes is [here](https://github.com/santoshn/softboundcets-34/blob/master/softboundcets-llvm-clang34/tools/softboundcets/main.cpp)
		* we might be able to move them to the newer version of LLVM 

- how does LLVM handle inline assembly?
	* [here](https://www.youtube.com/watch?v=MeB7Dp3G2UE) they have a video about it 

### How do you get call graph of a distributed system 
- distributed tracing
- paper
	+ [Canopy](https://people.mpi-sws.org/~jcmace/papers/kaldor2017canopy.pdf)
- Do we really need distributed tracting built by other people?
	+ we can build our own
    * at the end of the function, collect the profile results, and send them out.
		* have a seperate server that collects the results of profiling from each serverless function execution
	
```json
{
  "function_exec_time": "0.5ms",
  "calls": [
    {
      "Caller":"A",
      "Callee":"B",
      "counts":"1" 
    },
    {
      "Caller":"A",
      "Callee":"C",
      "counts":"2"
    } 
  ] 
}

``` 

### Other open-source compilers
- [Numba](https://numba.pydata.org/) for Python
	+ [github](https://github.com/numba/numba?tab=readme-ov-file)
- [Rustc](https://github.com/rust-lang/rust/tree/master/compiler) for Rust
	+ [Here](https://rust-lang.zulipchat.com/#narrow/stream/187780-t-compiler.2Fwg-llvm/topic/.E2.9C.94.20Running.20Custom.20LLVM.20Pass/near/320275483) is a post about how to write LLVM's optimization passes for Rustc 
- [C# can also yeilds LLVM IR](https://en.wikipedia.org/wiki/LLVM#:~:text=Originally%20implemented%20for%20C%20and,ActionScript%2C%20Ada%2C%20C%23%20for%20.)

### About DeathStarBench 
- the RPC in [DeathStarBench](https://github.com/delimitrou/DeathStarBench/blob/master/socialNetwork/gen-cpp/UniqueIdService.h#L224)
- I'm just curious about how to use Apache thrift Compiler
	* apache thrift will automatically generate server template for your service
	* then you can decide the 
	* [here](https://github.com/apache/thrift/tree/master/tutorial/cpp) is an example
- If we want to convert C++ code to C, we need some good JSON parser in C
	+ the [json parser library](https://www.json.org/json-en.html) in C
	+ <strong>another useful json parser</strong> [jsmn](https://github.com/zserge/jsmn/tree/master)

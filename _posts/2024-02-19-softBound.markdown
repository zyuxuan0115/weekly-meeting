---
layout: post
title:  "2024-02-19 SoftBound"
date:   2024-02-19 1:53:46 -0500
categories: serverless functions
---

### Open source serverless platform that supports C/C++
We don't need the FaaS runtime written in C/C++, we only need the serverless functions written in C/C++
#### OpenFaaS
- [openfaas](https://github.com/openfaas): there is no tutorial about how to run openfaas at all
	+ [install OpenFaas](https://gcore.com/learning/create-serverless-functions-with-openfaas/)
	+ In 2018 you create FaaS platform with openfaas in [this way](https://medium.com/@pavithra_38952/openfaas-on-docker-440541d635a2)
	+ deploy existing containers to OpenFaaS [webpage](https://www.openfaas.com/blog/porting-existing-containers-to-openfaas/)
	+ [docker support](https://docs.openfaas.com/languages/dockerfile/)
	+ [older version](https://www.digitalocean.com/community/tutorials/how-to-install-and-secure-openfaas-using-docker-swarm-on-ubuntu-16-04)
  + the example of starting the OpenFaaS
    * [Deploy OpenFaaS](https://ericstoekl.github.io/faas/deployment/swarm/#deploy-openfaas)
    * [Deploy OpenFaaS to Docker Swarm](https://ericstoekl.github.io/faas/deployment/swarm/)
  + [install OpenFaas](https://slopezza.medium.com/openfaas-installation-and-first-python-function-part-i-fa429053e8df): another example about why ironFunctions is slow.

![s1](/assets/2024-02-19/s1.webp)

#### the older example of starting the OpenFaaS
- The older version of [OpenFaaS' manual](https://ericstoekl.github.io/faas/)
	+ I think it works with the version with `802461dd025a8cec4784a2e40c7f4d0db501a2c3`
- [Deploy OpenFaaS to Docker Swarm](https://ericstoekl.github.io/faas/deployment/swarm/)
  + [Deploy OpenFaaS](https://ericstoekl.github.io/faas/deployment/swarm/#deploy-openfaas)
    * [the docker compose of docker swarm]()
	+ [FaaS-swarm](https://github.com/openfaas/faas-swarm/)
- tried several times but seems like it doesn't work


#### ironFunctions 
- [ironFunction](https://github.com/iron-io)
	+ [run IronFunction as a scheduler on top of Docker Standalone Swarm cluster](https://github.com/iron-io/functions/tree/master/docs/operating/docker-swarm)
    * [scheduling service on a docker swarm mode cluster](https://semaphoreci.com/community/tutorials/scheduling-services-on-a-docker-swarm-mode-cluster)
	+ [Protect the Docker daemon socket](https://docs.docker.com/engine/security/protect-access/)
		* that's why we need to set `DOCKER_TLS_VERIFY`, `DOCKER_HOST`, `DOCKER_CERT_PATH` etc. 
    * also there is a [stackoverflow post for this](https://stackoverflow.com/questions/38286564/docker-tls-verify-docker-host-and-docker-cert-path-on-ubuntu)
	+ the nice thing of `ironFunction`
		* it convert the user request into `STDIN`, and the output to user only need to be set to `STDOUT`

### SoftBound
- [paper](https://llvm.org/pubs/2009-06-PLDI-SoftBound.pdf) / [github repo](https://github.com/santoshn/softboundcets-34)

#### SoftBound for llvm3.4
- how to compile the code with SoftBound

```bash
clang -fsoftboundcets test.c -o test -L<git_repo>/softboundcets-lib -lm -lrt
```

- SoftBound doesn't take IR as the input
	+ but they still have the code of the LLVM pass
		* it's at [here](https://github.com/santoshn/softboundcets-34/tree/master/softboundcets-llvm-clang34/lib/Transforms/SoftBoundCETS) and [here](https://github.com/santoshn/softboundcets-34/tree/master/softboundcets-llvm-clang34/include/llvm/Transforms/SoftBoundCETS)
		* the other of the passes is [here](https://github.com/santoshn/softboundcets-34/blob/master/softboundcets-llvm-clang34/tools/softboundcets/main.cpp)

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

#### Ported SoftBound to llvm10
- [github code](https://github.com/zyuxuan0115/faas-cpp-test/tree/main/SoftBound-llvm10)

- how does LLVM handle inline assembly?
	* [here](https://www.youtube.com/watch?v=MeB7Dp3G2UE) they have a video about it 

### Get the call graph of a distributed system 
- distributed tracing
	+ [Canopy](https://people.mpi-sws.org/~jcmace/papers/kaldor2017canopy.pdf)
	+ [jaeger](https://www.jaegertracing.io/): used by DeathStarBench
		* [github](https://github.com/jaegertracing)
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
	  * [example code of jsondump](https://github.com/zserge/jsmn/blob/master/example/jsondump.c)
- Stateful serverless functions;
	+ disable all the writes

---
layout: post
title:  "2024-02-14 SoftBound"
date:   2024-02-14 1:53:46 -0500
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

### Light-weight contexts
- [paper](https://www.usenix.org/system/files/conference/osdi16/osdi16-litton.pdf) / [webpage](https://www.cs.umd.edu/projects/lwc/)
- FreeBSD [github](https://github.com/freebsd/freebsd-src) / [version](https://docs.freebsd.org/en/books/porters-handbook/versions/)
- can freebsd running in docker? 
	* cannot directly run a freebsd os in docker on Linux, from [web](https://stackoverflow.com/questions/33864142/can-freebsd-be-run-inside-docker) 
	* can run on a virtual machine inside docker [Youtube](https://www.youtube.com/watch?v=9-BowuxQrhE)

![d2](/assets/2024-02-02/d2.png)

- FreeBSD can be running in docker on a FreeBSD system
	- the webpage for [how to run docker on FreeBSD](https://wiki.freebsd.org/Docker) 
- the commit hash we are going to use is around this version:

```
85365dfcbf28 - Andrew Rybchenko, Wed Dec 28 10:40:21 2016 +0000 : sfxge(4): cleanup: remove trailing whitespace
```

- how do we use lwc to guarantee `isolation` when merging serverless func?
	+ assume we have 2 serverless functions A and B
	+ in `main`:
		* initialize serverless function A
		* `lwCreate` context for A
		* `if caller != -1 then run function call A`
		* erase the sensitive data in A
		* `lwRestrict` change the permission of the root lwc, so the root lwc won't have access to A's lwc.
		* then in the for loop
			- lwSwitch to A's lwc
			- lwSwitch to B's lwc
		* <strong>in the same way create function B's lwc</strong>
	+ on thing that need us to pay attention is
		* we need to use `lwRestrict` or `lwOverlay` to change the permission of memory region that stores the arguments we want to pass from A to B. 			 


### How do you get call graph of a distributed system 
- distributed tracing



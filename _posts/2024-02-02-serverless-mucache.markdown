---
layout: post
title:  "2024-02-02 FaaS + MuCache"
date:   2024-02-02 1:53:46 -0500
categories: serverless functions
---

### Other C++ serverless platform
We don't need the FaaS runtime written in C/C++, we only need the serverless functions written in C/C++
- [google functions framework C++](https://github.com/GoogleCloudPlatform/functions-framework-cpp)
	+ [this page](https://github.com/GoogleCloudPlatform/functions-framework-cpp/tree/main/examples/howto_use_legacy_code) shows an example about how to build C++ serverless functions using google's function-framework-cpp
	+ [build docker image and push to google's repo](https://cloud.google.com/artifact-registry/docs/docker/pushing-and-pulling)
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
- [Fn](https://fnproject.io/): tried but doesn't work with C++ serverless functions
  + [develop with multiple context](https://fnproject.io/tutorials/basics/UsingContexts/)
	+ [create a function from a Docker image](https://fnproject.io/tutorials/ContainerAsFunction/#FunctionDefinition)
- [nightcore](https://github.com/ut-osa/nightcore): acutally support call functions on different servers
	+ ![s1](/assets/2024-02-02/d1.png)
	+ in their foo calls bar [example](https://github.com/ut-osa/nightcore/tree/asplos-release/examples/c)

```c++
context->append_output_fn(context->caller_context,
                          output_buffer, strlen(kOutputPrefix) + bar_output_length);
```

- [ironFunction](https://github.com/iron-io)
	+ [run IronFunction as a scheduler on top of Docker Standalone Swarm cluster](https://github.com/iron-io/functions/tree/master/docs/operating/docker-swarm)
	+ [Protect the Docker daemon socket](https://docs.docker.com/engine/security/protect-access/)
		* that's why we need to set `DOCKER_TLS_VERIFY`, `DOCKER_HOST`, `DOCKER_CERT_PATH` etc. 
	+ the nice thing of `ironFunction`
		* it convert the user request into `STDIN`, and the output to user only need to be set to `STDOUT`

### SoftBound paper
- [paper](https://llvm.org/pubs/2009-06-PLDI-SoftBound.pdf) / [github repo](https://github.com/santoshn/softboundcets-34)
- SoftBound doesn't take the generated IR as the input.
	+ this is due to the fact that version of LLVM is too old 

### How do you get call graph of a distributed system 
- distributed tracing

### Light-weight contexts
- [paper](https://www.usenix.org/system/files/conference/osdi16/osdi16-litton.pdf) / [webpage](https://www.cs.umd.edu/projects/lwc/)
- FreeBSD [github](https://github.com/freebsd/freebsd-src) / [version](https://docs.freebsd.org/en/books/porters-handbook/versions/)

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


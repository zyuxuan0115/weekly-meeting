---
layout: post
title:  "2023-9-30 nightcore benchmark"
date:   2023-9-30 1:53:46 -0500
categories: serverless functions
---
### nightcore benchmark
- According to Haoran, in nightcore benchmark, it has the controller
- [github repo](https://github.com/ut-osa/nightcore-benchmarks/tree/master), [nightcore paper](https://www.cs.utexas.edu/users/witchel/pubs/jia21asplos-nightcore.pdf)
	+ the evaluation runs on aws. but we (I) only have cloudlab
- in nightcore benchmark's [exp_helper](https://github.com/ut-osa/nightcore-benchmarks/blob/master/scripts/exp_helper#L148) script
	+ a function called `setup_docker_swarm_for_machines`
	+ another function called `start_machines_main`
		* in `start_machine_main`, the [config.json](https://github.com/ut-osa/nightcore-benchmarks/blob/master/experiments/hipstershop_4node/config.json) is called. 
- A little bit about [docker](https://en.wikipedia.org/wiki/Docker_(software))
	+ things about docker's [swarm](https://docs.docker.com/engine/swarm/)
		* Swarm mode is an advanced feature for managing a cluster of Docker daemons.
		* Decentralized design: people can deploy both kinds of nodes, managers and workers, using the Docker Engine.
	+ to install docker: 
		* `apt install docker.io`
	+ some useful docker instructions
		* `docker build`: Build an image from a Dockerfile
			- `-t, --tag list`: Name and optionally a tag in the 'name:tag' format
			- `-f, --file string`: Name of the Dockerfile (Default is 'PATH/Dockerfile')
		* `docker push`: 
			- Usage:	docker push [OPTIONS] NAME[:TAG]
			- Push an image or a repository to a registry 
		* <strong>docker swarm</strong>
			- tutorial: [3 networked host machines](https://docs.docker.com/engine/swarm/swarm-tutorial/#three-networked-host-machines)

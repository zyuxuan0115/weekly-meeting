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
		* in `start_machine_main`, the [config.json](https://github.com/ut-osa/nightcore-benchmarks/blob/master/experiments/hipstershop_4node/config.json) is read.
		* this file is important because it tells us how to config manager and worker nodes for <strong>docker swarm</strong> 
- A little bit about [docker](https://en.wikipedia.org/wiki/Docker_(software))
	+ things about docker's [swarm](https://docs.docker.com/engine/swarm/)
		* Swarm mode is an advanced feature for managing a cluster of Docker daemons.
		* Decentralized design: people can deploy both kinds of nodes, managers and workers, using the Docker Engine.
	+ to install docker: 
		* `apt install docker.io`
	+ useful docker instructions for creating a docker image
		* `docker build`: Build an image from a Dockerfile
			- `-t, --tag list`: Name and optionally a tag in the 'name:tag' format
			- `-f, --file string`: Name of the Dockerfile (Default is 'PATH/Dockerfile')
		* `docker push`: 
			- Usage:	docker push [OPTIONS] NAME[:TAG]
			- Push an image or a repository to a registry
		* [how to write a docker file](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/) 
	+ useful docker instructions for using <strong>docker swarm </strong>
		* tutorial: [3 networked host machines](https://docs.docker.com/engine/swarm/swarm-tutorial/#three-networked-host-machines)
		* `docker stack`: Docker stack is ignoring “build” instructions. It need pre-built images to exist.
			- example: `docker stack deploy -c docker-compose.yml somestackname`
			- need a `docker-compose.yml` file. in nightcore-benchmark, the yml file is [here](https://github.com/ut-osa/nightcore-benchmarks/blob/master/experiments/hipstershop_4node/docker-compose.yml)
			- more about `docker stack deploy` is [here](https://docs.docker.com/engine/reference/commandline/stack_deploy/)

---
layout: post
title:  "2023-9-30 nightcore benchmark"
date:   2023-9-30 1:53:46 -0500
categories: serverless functions
---
### Nightcore Benchmark
- According to Haoran, in nightcore benchmark, it has the controller
- [github repo](https://github.com/ut-osa/nightcore-benchmarks/tree/master), [nightcore paper](https://www.cs.utexas.edu/users/witchel/pubs/jia21asplos-nightcore.pdf)
	+ the evaluation runs on aws. but we (I) only have cloudlab
- before everything runs it builds the docker image of nightcore
- In `nightcore-benchmark/experiments/<benchmark-name>/run_all.sh`
	+ it first runs [exp_helper](https://github.com/ut-osa/nightcore-benchmarks/blob/master/scripts/exp_helper#L148) with argument `start-machine` to create a <strong>docker swarm</strong>
	+ then it writes to `machine_infos` about all machines information in the created docker swarm 
		* in nightcore benchmark's [exp_helper](https://github.com/ut-osa/nightcore-benchmarks/blob/master/scripts/exp_helper#L148) script
			- function `start_machines_main` calls `setup_docker_swarm_for_machines`
			- in `start_machine_main`, the [config.json](https://github.com/ut-osa/nightcore-benchmarks/blob/master/experiments/hipstershop_4node/config.json) is read.
				+ this file is important because it tells us how to config manager and worker nodes for <strong>docker swarm</strong> 
	+ then it invokes `run_once` multiple times in order to
		* run `docker stack deploy` on manager
			- `docker stack deploy -c docker-compose.yml -c docker-compose-placement.yml <stack name>`
			- manager deploys multiple workers that run nightcore <strong>(?)</strong>
		* on client host, run the workloads' script
			- [hipstershop benchmark](https://github.com/ut-osa/nightcore-benchmarks/blob/master/experiments/hipstershop_4node/run_once.sh#L62)
	+ according to the [docker-compose.yml](https://github.com/ut-osa/nightcore-benchmarks/blob/master/experiments/hipstershop_4node/docker-compose.yml)
		* nightcore-engine is only declared once
		* but in [run_once.sh](https://github.com/ut-osa/nightcore-benchmarks/blob/master/experiments/hipstershop_4node/run_once.sh#L19), it listed 4 engine nodes.
			- does it mean docker will automatically create 4 engine nodes, because from `docker swarm` + [config.sh](https://github.com/ut-osa/nightcore-benchmarks/blob/master/experiments/hipstershop_4node/config.json) 4 engine nodes has already been created?

```yml
version: "3.8"
services:
  nightcore-engine:
    image: zjia/nightcore:asplos-ae
    entrypoint:
      - /nightcore/engine
      - --gateway_addr=nightcore-gateway
      - --root_path_for_ipc=/tmp/nightcore/ipc
      - --func_config_file=/tmp/nightcore/func_config.json
      - --num_io_workers=1
      - --gateway_conn_per_worker=32
      - --disable_monitor
      - --instant_rps_p_norm=0.8
    depends_on:
      - nightcore-gateway
    volumes:
      - /mnt/inmem/nightcore:/tmp/nightcore
      - /sys/fs/cgroup:/tmp/root_cgroupfs
    environment:
      - FAAS_CGROUP_FS_ROOT=/tmp/root_cgroupfs

  nightcore-gateway:
    image: zjia/nightcore:asplos-ae
    entrypoint:
      - /nightcore/gateway
      - --listen_addr=0.0.0.0
      - --http_port=8080
      - --grpc_port=50051
      - --func_config_file=/tmp/nightcore/func_config.json
      - --num_io_workers=8
      - --max_running_requests=24
      - --lb_pick_least_load
    ports:
      - 8080:8080
    volumes:
      - /tmp/nightcore_config.json:/tmp/nightcore/func_config.json

  frontend-api-home:
    image: zjia/nightcore-hipstershop:frontend-api-asplos-ae
    command:
      - --func_id=10
      - --root_path_for_ipc=/tmp/nightcore/ipc
      - --fprocess_output_dir=/tmp/nightcore/output
    env_file:
      - common.env
    environment:
      - GOGC=1000
      - FAAS_GO_MAX_PROC_FACTOR=8
      - SWARM_TASK_SLOT={{.Task.Slot}}
    volumes:
      - /mnt/inmem/nightcore:/tmp/nightcore
    depends_on:
      - nightcore-engine
```

### Some thoughts about making Foo on server1 and Bar on Server2
- functions are listed in nightcore's C example's [func_config.json](https://github.com/ut-osa/nightcore/blob/asplos-release/examples/c/func_config.json) file.
- In nightcore-benchmark's [script](https://github.com/ut-osa/nightcore-benchmarks/blob/master/experiments/hipstershop_4node/run_once.sh#L42), the same file is copied to the engine nodes. 
	+ if we copy different `func_config.json` to different engine nodes, we are able to make `Foo` only on server1, `Bar` only on server2.
	+ then still send the `curl -X POST -d "Hello" http://<gateway>/function/Foo`
	+ we will see `From function Bar: Hello, World`, and measure the time spent from `curl` being sent to the message `From function Bar:` being received.
```json
[
    { "funcName": "Foo", "funcId": 1, "minWorkers": 2, "maxWorkers": 2 },
    { "funcName": "Bar", "funcId": 2, "minWorkers": 2, "maxWorkers": 2 }
]
```

### About [docker](https://en.wikipedia.org/wiki/Docker_(software))
- things about docker's [swarm](https://docs.docker.com/engine/swarm/)
	+ Swarm mode is an advanced feature for managing a cluster of Docker daemons.
	+ Decentralized design: people can deploy both kinds of nodes, managers and workers, using the Docker Engine.
- to install docker: 
	+ `apt install docker.io`
- useful docker instructions for creating a docker image
	+ `docker build`: Build an image from a Dockerfile
		* `-t, --tag list`: Name and optionally a tag in the 'name:tag' format
		* `-f, --file string`: Name of the Dockerfile (Default is 'PATH/Dockerfile')
	+ [how to write a docker file](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/) 
- useful docker instructions for <strong>docker swarm </strong>
	+ tutorial: [3 networked host machines](https://docs.docker.com/engine/swarm/swarm-tutorial/#three-networked-host-machines)
	+ `docker stack`: Docker stack is ignoring “build” instructions. It need pre-built images to exist.
		* example: `docker stack deploy -c docker-compose.yml somestackname`
		* need a `docker-compose.yml` file. in nightcore-benchmark, the yml file is [here](https://github.com/ut-osa/nightcore-benchmarks/blob/master/experiments/hipstershop_4node/docker-compose.yml)
		* more about `docker stack deploy` is [here](https://docs.docker.com/engine/reference/commandline/stack_deploy/)
		* more about `docker-compose.yml` file is [here](https://docs.docker.com/compose/gettingstarted/)

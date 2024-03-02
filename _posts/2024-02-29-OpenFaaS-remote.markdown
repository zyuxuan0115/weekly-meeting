---
layout: post
title:  "2024-02-29 OpenFaaS remote + Kubernete"
date:   2024-02-29 1:53:46 -0500
categories: serverless functions
---

## OpenFaaS
- [openfaas](https://github.com/openfaas): there is no tutorial about how to run openfaas at all
	+ deploy existing containers to OpenFaaS [webpage](https://www.openfaas.com/blog/porting-existing-containers-to-openfaas/)
	+ [docker support](https://docs.openfaas.com/languages/dockerfile/)
  + [how to deploy functions using OpenFaaS](https://gcore.com/learning/create-serverless-functions-with-openfaas/) - super useful
	+ [Set-up OpenFaaS with Kubernetes](https://github.com/openfaas/workshop/blob/master/lab1b.md#run-on-digitaloceans-kubernetes-service) - super useful
	+ [first OpenFaaS Function with Python](https://docs.openfaas.com/tutorials/first-python-function/) - super useful

### change docker permission

```bash
> sudo groupadd docker 
> sudo gpasswd -a $USER docker
> newgrp docker 
> sudo chmod -R 777 /users/zyuxuan/.docker
```
	
### deploy OpenFaaS on local clusters
- [tutorial](https://docs.openfaas.com/deployment/kubernetes/)
- steps for deploying OpenFaaS on the local cluster
  + first install Docker, golang
	+ install [arkade](https://github.com/alexellis/arkade)
  + install [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/)
	+ then install [KinD](https://kind.sigs.k8s.io/) 
    * I [installed from a released binary](https://kind.sigs.k8s.io/docs/user/quick-start/#installing-from-release-binaries)
  + create a cluster quickly
    * `kind create cluster`	
	+ [faas-cli](https://docs.openfaas.com/cli/install/): `curl -sSL https://cli.openfaas.com | sudo -E sh`
  + run openfaas
    * `arkade install openfaas --load-balancer`
      - Note: the `--load-balancer` flag has a default of false, so by passing the flag, the installation will request one from your cloud provider.
    * to verify if you successfully run openfaas: 
      - `kubectl -n openfaas get deployments -l "release=openfaas, app=openfaas"`

```bash
# Forward the gateway to your machine
> kubectl rollout status -n openfaas deploy/gateway
> kubectl port-forward -n openfaas svc/gateway 8080:8080 &

# If basic auth is enabled, you can now log into your gateway:
> PASSWORD=$(kubectl get secret -n openfaas basic-auth -o jsonpath="{.data.basic-auth-password}" | base64 --decode; echo)
> echo -n $PASSWORD | faas-cli login --username admin --password-stdin
```

- delete things in k3s
  + a deployment in k3s
    * `kubectl -n openfaas delete deployment gateway` 
  + a node in k3s
    * `kubectl delete node <node name>`

- to update a function
	+ first delete a function and then run `faas-cli remove <function name>`
  + then redeploy it

- to deploy the service

```bash
> faas-cli deploy -f ./hello-rust.yml
```

- to invoke the serverless function

```bash
> curl 127.0.0.1:8080/function/hello-rust -d "This is Yuxuan."
```




### deploy OpenFaaS on remote cluster with multiple machines
- [k3sup](https://github.com/alexellis/k3sup)
  + the most important part is [Create a multi-master](https://github.com/alexellis/k3sup?tab=readme-ov-file#create-a-multi-master-ha-setup-with-embedded-etcd) and [Join some agents to your Kubernetes server](https://github.com/alexellis/k3sup?tab=readme-ov-file#-join-some-agents-to-your-kubernetes-server)

- create server

```bash
> export SERVER_IP=192.168.0.100
> export USER=zyuxuan
> k3sup install --ip $SERVER_IP --user $USER
```

- create agent

```bash
> export AGENT_IP=192.168.0.101
> export USER=zyuxuan
> k3sup join --ip $AGENT_IP --server-ip $SERVER_IP --user $USER
```

- try to access

```bash
> export KUBECONFIG=`pwd`/kubeconfig
> kubectl get node
```

- run openfaas

```bash
> arkade install openfaas --load-balancer
```

- fowward the gateway to the machine

```bash
# Forward the gateway to your machine
> kubectl rollout status -n openfaas deploy/gateway
> kubectl port-forward -n openfaas svc/gateway 8080:8080 &
# If you're using a managed cloud Kubernetes service then get the 
# LoadBalancer's IP address or DNS entry from the EXTERNAL-IP field 
# from the command below.
> kubectl get svc -o wide gateway-external -n openfaas
# If basic auth is enabled, you can now log into your gateway:
> PASSWORD=$(kubectl get secret -n openfaas basic-auth -o jsonpath="{.data.basic-auth-password}" | base64 --decode; echo)
> echo -n $PASSWORD | faas-cli login --username admin --password-stdin
```

- if a function is invoked within kubernetes cluster, to call the function, the REST api is:
  + `http://gateway.openfaas.svc.cluster.local.:8080`
  + reference is [here](https://docs.openfaas.com/reference/rest-api/#:~:text=Functions%20can%20be%20invoked%20by,path%20to%20the%20gateway%20URL.&text=If%20no%20namespace%20is%20specified,%2Fasync%2Dfunction%2FNAME.)

#### to stop a service

```bash
> kubectl -n openfaas get deployments -l "release=openfaas, app=openfaas"
> kubectl -n openfaas delete deployment <name>
> kubectl get nodes
> kubectl drain --ignore-daemonsets --delete-emptydir-data <node name>
> kubectl delete <node name>
> sudo lsof -i -P -n | grep LISTEN
> kill <PID of kubectl which uses port 8080>
```

## Kubernetes
- [Kubernetes Components](https://kubernetes.io/docs/concepts/overview/components/)
- [Kubernetes cluster has its cluster DNS](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/)
	+ The Domain Name System (DNS) is the phonebook of the Internet. DNS translates domain names to IP addresses so browsers can load Internet resources.
- curious about what is a agent in kubernete
	+ [the architecture of k3s](https://docs.k3s.io/architecture)


## C + SoftBound + Rust
- works well

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



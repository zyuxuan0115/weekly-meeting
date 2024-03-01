---
layout: post
title:  "2024-02-29 OpenFaaS remote"
date:   2024-02-29 1:53:46 -0500
categories: serverless functions
---

## OpenFaaS
- [openfaas](https://github.com/openfaas): there is no tutorial about how to run openfaas at all
	+ deploy existing containers to OpenFaaS [webpage](https://www.openfaas.com/blog/porting-existing-containers-to-openfaas/)
	+ [docker support](https://docs.openfaas.com/languages/dockerfile/)
  + [how to deploy functions using OpenFaaS](https://gcore.com/learning/create-serverless-functions-with-openfaas/) - super useful
	+ [first OpenFaaS Function with Python](https://docs.openfaas.com/tutorials/first-python-function/)

### change docker permission

```bash
> sudo groupadd docker 
> sudo gpasswd -a $USER docker
> newgrp docker 
> sudo chmod -R 777 /users/zyuxuan/.docker
```
	
### the current version of OpenFaaS
- [tutorial](https://docs.openfaas.com/deployment/kubernetes/)
- steps for deploying OpenFaaS on the local cluster
  + first install Docker, golang
	+ install [arkade](https://github.com/alexellis/arkade)
  + install [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/)
	+ then install [KinD](https://kind.sigs.k8s.io/) 
    * I [installed from a released binary](https://kind.sigs.k8s.io/docs/user/quick-start/#installing-from-release-binaries)
  + create a cluster quickly
    * `kind create cluster`	
	+ [install cli](https://docs.openfaas.com/cli/install/)
    * `curl -SLsf https://get.arkade.dev/ | sudo sh`
  + run openfaas
    * `arkade install openfaas`
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

- delete a deployment in kubectl
  + `kubectl -n openfaas delete deployment gateway`

- to update a function
	+ first delete a function and then run `faas-cli remove <function name>`
  + then redeploy it

- to deploy the service

```bash
> faas-cli deploy -f ./hello-rust.yml
```

- to invoke the serverless function

```
> curl 127.0.0.1:8080/function/hello-rust -d "This is Yuxuan."
```

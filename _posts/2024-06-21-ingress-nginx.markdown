---
layout: post
title:  "2024-06-20 ingress nginx"
date:   2024-06-20 1:53:49 -0500
categories: serverless
---

## Kubernetes Ingress

### How Ingress-Nginx works
- <strong> [Kubernetes Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/)</strong>

![ingress](/assets/2024-06-21/d1.png)


- The real correct way to install [ingress-nginx](https://github.com/kubernetes/ingress-nginx/blob/main/docs/deploy/index.md)
  + expecially install to a <strong>bare metal cluster</strong>.
  + `kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.10.1/deploy/static/provider/baremetal/deploy.yaml`

### Some useful kubectl commands

```bash
# Get the detail of a pod for the ingress-nginx
kubectl -n ingress-nginx get pods
kubectl -n ingress-nginx describe pod <pod name>
# If the service type is NodePort, to get the port of the service
kubectl -n openfaas describe svc gateway-external
# get the node port of the service
NODE_PORT="$(kubectl get services/gateway-external -n openfaas -o go-template='{{(index .spec.ports 0).nodePort}}')"
echo $NODE_PORT
# get ingress
kubectl get ingress -n openfaas
# get all namespace in kubernetes
kubectl get ns
kubectl describe ns
# see the log of a pod
kubectl logs <pod-name> 
```

- Can also set the type of service to be `LoadBalancer`
  + this way, the port of nginx server will be fixed to 80 (http) & 443 (https)
  + you must explicitly assign an external IP for it
    * reference is [here](https://paul-boone.medium.com/kubernetes-loadbalancer-ip-stuck-in-pending-6ddea72b8ff5)
  + when creating `LoadBalancer` type of service, sometimes it will fail. 

```yaml
apiVersion: v1
kind: Service
metadata:
  name: k8salliance-service
spec:
  type: LoadBalancer
  selector:
    app: k8salliance-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 9000
  externalIPs:
  - "34.74.203.201"
``` 

- If you use `NodePort` as the service type, use the following command to get the port of nginx

```bash
NODE_PORT="$(kubectl get svc/ingress-nginx-controller -n ingress-nginx -o go-template='{{(index .spec.ports 0).nodePort}}')"
```

### OpenFaaS's log vs Ingress-Nginx's log
![no-ingress](/assets/2024-06-21/d2.png)

#### OpenFaaS  (when function make RPCs)
  + cannot see the caller info
  + `kubectl logs <pod's name>` 

![s2](/assets/2024-06-21/s2.png)

#### Ingress-Nginx (when function make RPCs)

![s3](/assets/2024-06-21/s3.png)

- How do we know it's caller's IP?

```bash
kubectl -n openfaas-fn get pods
kubectl -n openfaas-fn describe pod <pod's name>
```

![s4](/assets/2024-06-21/s4.png)


### Open-tracing for ingress-nginx
[use open-telemetry for ingress-nginx](https://kubernetes.github.io/ingress-nginx/user-guide/third-party-addons/opentelemetry/)

![s1](/assets/2024-06-21/s1.png)

[Tempo query: Query with TraceQL](https://grafana.com/docs/tempo/latest/traceql/)

### other topics related to OpenFaaS or Kubernetes
- [install openfaas using helm](https://artifacthub.io/packages/helm/openfaas/openfaas)
- [Kubernetes wait for secret to be created](https://stackoverflow.com/questions/71384532/kubernetes-wait-for-secret-to-be-created)

### OpenFaaS's function ingress using nginx
- example about how to add ingress for function: [OpenFaaS's function ingress operator](https://github.com/openfaas/ingress-operator)
- openfaas's yaml files is [here](https://github.com/openfaas/faas-netes/tree/master/chart/openfaas/templates)

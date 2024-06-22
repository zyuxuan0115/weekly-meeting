---
layout: post
title:  "2024-06-20 ingress nginx"
date:   2024-06-20 1:53:49 -0500
categories: serverless
---

### Kubernetes Ingress

- [Kubernetes Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/)

![ingress](/assets/2024-06-21/ingress.svg)

- [Install the ingress-nginx IngressController locally](https://docs.openfaas.com/tutorials/local-kind-ingress/)
- [install openfaas using helm](https://artifacthub.io/packages/helm/openfaas/openfaas)

- The real correct way to install [ingress-nginx](https://github.com/kubernetes/ingress-nginx/blob/main/docs/deploy/index.md)
  + expecially install to a <strong>bare metal cluster</strong>.
  + `kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.10.1/deploy/static/provider/baremetal/deploy.yaml`


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
```

- Can also set the type of service to be `LoadBalancer`
  + this way, the port of nginx server will be fixed to 80 (http) & 443 (https)
  + If you don't want your service's external-IP field to be pending (this step is not necessary)
    * you can explicitly assign an external IP for it
    * reference is [here](https://paul-boone.medium.com/kubernetes-loadbalancer-ip-stuck-in-pending-6ddea72b8ff5)

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

### Open-tracing for ingress-nginx
[use open-telemetry for ingress-nginx](https://kubernetes.github.io/ingress-nginx/user-guide/third-party-addons/opentelemetry/)

![s1](/assets/2024-06-21/s1.png)

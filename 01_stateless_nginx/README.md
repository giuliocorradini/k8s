# 01. Stateless NGINX

Goal: **deploy a stateless Nginx instance using Kuberenetes Deployment objects**

## Deployment object

A deployment describes a set of pods and how they should behave while running.

It can be described using a yaml file.

`kubectl apply` can be used to apply a configuration (described using yaml or json) to a resource, that must be specified.

Our Deployment resource object is specified in the file, if it's non-existent on the cluster it will be created.

The configuration file can also be retrieve from an http server by passing an URL.

```
kubectl apply -f https://k8s.io/examples/application/deployment.yaml
```

The `describe` command gives information about the current deployment (that runs as soon as its applied).

```
kubectl describe deployment nginx-deployment
kubectl describe <object_type> <object_name>
```

ReplicaSet is found more than once in the description, what is it?
[Official docs](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/)

A ReplicaSet is a set of Pods that is guaranteed to be running at any given time.

To inspect which pods are running you can issue `kubectl get pods -l <label>`

From the previous command we get the Pod Template label, which is `app=nginx`.

Remember that Pods are the smallest deployable unit on k8s, and are a collection of containers, shared volumes and networks, that runs together.

## Deployment configuration analysis

`apiVersion`: different versions of the k8s API may exist, and users can create their own extension to Kubernetes and extend the API too.
For this purpose API groups were implemented, and are enabled by default.

The API groups is specified in the `apiVersion` field, using the following syntax: `<group>:<version>`.
If group is not specified, the version refers to the **core** group.

When using ControllerRevision and StatefulSet objects, apps/v1 API group is required.

[Which Kubernetes apiVersion should I use?](https://matthewpalmer.net/kubernetes-app-developer/articles/kubernetes-apiversion-definition-guide.html)

`kind`: what kind of object the document is describing

`metadata > name`: a name for the object. If you edit the configuration and apply it, the current deployment will be updated based on its name.

`selector`: used to match the ReplicaSet against a set of labels

`matchLabels` (labels): matches a list of key-value pairs. Labels don't provide uniqueness and are used to organise Pods or other resources (names and UID provide uniquess instead).

after updating the replica instances to 4:

```
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  50m   deployment-controller  Scaled up replica set nginx-deployment-66b6c48dd5 to 2
  Normal  ScalingReplicaSet  10s   deployment-controller  Scaled up replica set nginx-deployment-66b6c48dd5 to 4
```

## Exposing the service

_Reference_

[Service - Official Docs](https://kubernetes.io/docs/concepts/services-networking/service/)

Kubernetes can expose a set of pods as a network service on the host. To do such thing we need to "expose a deployment"
thus creating a service.

```bash
kubectl expose deployment <deployment-name> --type=LoadBalancer --name=<service-name>
```

Services can be used to expose a set of pods to the outside world, or to another set of pods (such as a backend to a
frontend worker).

If we inspect running services:

```bash
kubectl get svc
```

we can see that our (nginx-deployment) service has a pending external IP address.

Since we're running on minikube, if we start `minikube tunnel` pending IP addresses for LoadBalancer services
are instantly resolved. _nginx-deployment_ is now exposed via 127.0.0.1

N.B. minikube tunnel will ask for **root permits** when publishing on protected ports (1-1000),
so don't forget to enter your password.

```
> kubectl describe svc nginx-deployment

Name:                     nginx-deployment
Namespace:                default
Labels:                   <none>
Annotations:              <none>
Selector:                 app=nginx
Type:                     LoadBalancer
IP Families:              <none>
IP:                       10.98.168.222
IPs:                      10.98.168.222
LoadBalancer Ingress:     127.0.0.1
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  31247/TCP
Endpoints:                172.17.0.6:80,172.17.0.7:80,172.17.0.8:80 + 1 more...
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```

Note that Port is the exposed port via which you can reach the LoadBalancer.

```bash
curl http://127.0.0.1:80
```

## Cleanup

1. Delete the service

```bash
kubectl delete svc nginx-deployment
```

2. Delete the deployment

Otherwise it will start the next time you fire the cluster (minikube)

```bash
kubectl delete deployment nginx-deployment
```

3. Stop minikube

```bash
minikube stop
```

or shut down your cluster.
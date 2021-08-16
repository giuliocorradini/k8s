# 02. k3s bringup

k3s is an open source flavour of kubernetes that runs with a small memory and CPU footprint.
It's therefore perfect for edge and embedded applications as well as for server deployment.

To get started with k3s on a Linux machine we install it using the following command:
(make sure you have curl installed)

```bash
curl -sfL https://get.k3s.io | sh -
```

Now k3s and kubernetes tools (kubectl) are installed on your system. To automatically start
k3s at startup enable its service via systemd:

```bash
systemctl enable k3s
```

## Remote access to the cluster

We need to remotely access this cluster using our local instance of `kubectl`.

[Accessing Clusters | Kubernetes](https://kubernetes.io/docs/tasks/access-application-cluster/access-cluster/)

[Configure Access to Multiple Clusters](https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/)

Following the official k3s guide, we download the k3s.yaml config file to access the cluster:

```bash
scp root@deb-k3s.local:/etc/rancher/k3s/k3s.yaml .
```

[Cluster Access](https://rancher.com/docs/k3s/latest/en/cluster-access/)

and we set the `KUBECONFIG` environment variable to route our kubectl commands directly to the remote cluster.

```bash
export KUBECONFIG=$KUBECONFIG:k3s.yaml
```

Before issuing any kubectl command we need to edit _k3s.yaml_ and change the cluster IP.

Let's see which nodes are currently running:

```bash
$ kubectl get nodes
NAME      STATUS   ROLES                  AGE     VERSION
deb-k3s   Ready    control-plane,master   3d21h   v1.21.3+k3s1
```

## Deploying and exposing a service on k3s

If we run *01_stateless_nginx* we can see that and external IP is not assigned to our service,
but it's currently running on port 31619 (that links to port 80).

K3s includes a LoadBalancer named Klipper that makes use of available host ports to expose services.

### Service creation policy

Klipper creates a worker Pod for every exposed service, that acts as a proxy between the external world
and your Deplyoment.

It's possible to run multiple services on the same node as long as they have different ports, otherwise
the Service LB will try to expose your service on a node with a free port.

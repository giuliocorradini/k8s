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

[Cluster Access](https://rancher.com/docs/k3s/latest/en/cluster-access/)

and we set the `KUBECONFIG` environment variable to route our kubectl commands directly to the remote cluster.

```bash
export KUBECONFIG=$KUBECONFIG:k3s.yaml
```
# 03 Multiple nodes cluster

From the official documentation of k8s regarding the architecture of a cluster:

*A node is a VM or a physical computer that serves as a worker machine in a Kubernetes cluster. Each node has a Kubelet, which is an agent for managing the node and communicating with the Kubernetes control plane. The node should also have tools for handling container operations, such as containerd or Docker. A Kubernetes cluster that handles production traffic should have a minimum of three nodes.*

[Cluster Introduction](https://kubernetes.io/docs/tutorials/kubernetes-basics/create-cluster/cluster-intro/)

We're currently deploying a single VM that acts as both the control plane and a node of the cluster. To add another node
we start by firing a new virtual machine.

Using a VM template, k3s is already installed. Otherwise run the usual script:

```bash
curl -sfL https://get.k3s.io | sh -
```

If you start with a pre-installed k3s instance, it won't be downloaded again by the script. Neat.

Now get the node token from the master node, located in `/var/lib/rancher/k3s/server/node-token`

Start k3s as agent and connect it to the cluster. Be careful to edit the hostname of your VM if you
start from the template, otherwise the new agent node will not connect.

```bash
k3s agent --server https://<ip>:6443 --token <token> --node-name <nodename>
```

[Building up k3s Cluster](https://dockerlabs.collabnix.com/beginners/install/raspberrypi3/setting-up-k3s-cluster.html)

## Automating with systemd

We can automatically start k3s agent at startup by writing a systemd script. Server IP, token and node name
are set from an environment file named `k3s-agent.service.env`.

Please edit the environment file before using it.
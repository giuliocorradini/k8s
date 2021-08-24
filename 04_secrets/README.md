# 04 Secrets

Secrets contain sensitive data that should be handled carefully (such as a database password, or
an API token) and not shipped with your containers.

There are different types of secrets available in kubernetes, such as SSH keys, Docker
Registry keys, basic authentication (username:password), TLS cert and key etc.

[Types of Secret | Kubernetes](https://kubernetes.io/docs/concepts/configuration/secret/#secret-types)

## Create secrets...

Let's create a generic secret (for the sake of this tutorial) named `database-access`, containing
a username and a password.

```bash
kubectl create secret generic database-access \
    --from-file=username=username.txt \
    --from-file=password=password.txt
```

`--from-file` accepts a keyname as its second token, and the filename as its third. Complete syntax:

`--from-file=[key]=[filename]`

Opaque (or generic) secrets are stored as a base64 encoded string. To retrieve your data you have to retrieve
the .data object of your secret.

```bash
kubectl get secret database-access -o jsonpath='{.data}'
```

The command will return a JSON object with your keys and your data encoded in base64. You may use the command
base64 to perform decoding.

## ...and use them

Secrets are consumed by pods as if they were normal file mounted on a special read-only filesystem, that
you can bind (or map) anywhere in your system.

Let's set "/secrets" as the path that will contain our secrets, and create a small Docker container that expose a
Python http server with directory navigation enabled. If secrets are files, this will let us inspect all available
secrets.

## Digression on image pull policy

When working with custom containers that are usually built from scratch on the developer's machine, problems may
arise when trying to deploy pods that make use of them.

How do Kubernetes behave with Docker container images?

[Images | Kubernetes Docs](https://kubernetes.io/it/docs/concepts/containers/images/)

Images names always refer to the public Docker registry, if not otherwise specified. K8s will automatically
search your image on hub.docker.com, but you may also specify a different public registry in the image name
such as `ghcr.io/my/k8s_secrets_server`.

Kubernetes can create a Pod template using a pre-pulled image (already in the node cache), for testing purposes
pre-pulled images can be used but they're more complex to deal with.

If we set the `imagePullPolicy` property on `Never`, k8s will automatically use a cached image. But this won't work
if we're using k3s or if our cluster has multiple nodes.

Instead we need to setup a local registry. We can start a Docker on the master node and push our image there.
In 05_dns we'll configure a custom DNS name for our registry for internal cluster usage, and we'll inspect the
included DNS service in k3s (CoreDNS).

### Start a registry daemon

Open a console on your master node and start a registry daemon

```
docker run -d -p 5000:5000 --restart=always --name registry registry:2
```

Now tag your image with `localhost:5000` (registry address) and push it.

```
docker tag k8s_secrets_server localhost:5000/server
docker push localhost:5000/k8s_secrets_server
```

The image should now be on our local registry and k8s should be able to pull it.

## Back to pod configuation

We now specify that our pod should use the image `localhost:5000/k8s_secrets_server` and we apply
the configuration.

K8s will start pulling the image from the local registry, if you have docker tools installed on your node
and you inspect the cached images (docker image list) you will see the freshly pulled image
(localhost:5000/k8s_secrets_server).

N.B. Kubernetes might not find locally built images even if they're in the cached registry. Always use an
external registry.

### Expose the pod through a service

```
kubectl expose pod secrets-server-pod --type=LoadBalancer --name=secpod
```

K3s integrated load balanacer (Klipper) will expose our service on the host node port. To get it
inspect available services using `get svc` and connect to it via a browser.

`http://192.168.178.35:31289/`

The webpage shows the content for the /secrets directory, which contains two files: username and password.
These files contains the raw data with username and password (unlike kubectl inspections, it's not base64 encoded).

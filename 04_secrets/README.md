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


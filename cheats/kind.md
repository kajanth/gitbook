# Kind

### Install

Check latest version: [https://github.com/kubernetes-sigs/kind/releases](https://github.com/kubernetes-sigs/kind/releases)

```text
wget https://github.com/kubernetes-sigs/kind/releases/download/v0.8.1/kind-linux-amd64
chmod +rx kind-linux-amd64
sudo mv kind-linux-amd64 /usr/local/bin/kind
```

## Cluster Management

### Create

Go to: [https://hub.docker.com/r/kindest/node/tags](https://hub.docker.com/r/kindest/node/tags)

Find the image tag you want to use, which will be the Kubernetes verison.

Get the image tag and sha256.

Create a config file, for example.

{% code title="kind.yaml" %}
```text
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
networking:
  apiServerAddress: 192.168.0.102 # DEFAULT 127.0.0.1
nodes:
- role: control-plane
  image: kindest/node:v1.17.5@sha256:ab3f9e6ec5ad8840eeb1f76c89bb7948c77bbf76bcebe1a8b59790b8ae9a283a
- role: worker
  image: kindest/node:v1.17.5@sha256:ab3f9e6ec5ad8840eeb1f76c89bb7948c77bbf76bcebe1a8b59790b8ae9a283a
```
{% endcode %}

Create the cluster.

```text
kind create cluster --name kind-001 --config ./kind.yaml
```

Alternatively, you can run:

```text
kind create cluster --name kind-001 --image kindest/node:v1.17.5@sha256:ab3f9e6ec5ad8840eeb1f76c89bb7948c77bbf76bcebe1a8b59790b8ae9a283a
```

### Delete

```text
kind delete cluster --name kind-001
```

## Kubeconfig

### Get kubeconfig

```text
kind get kubeconfig --name kind-001
```


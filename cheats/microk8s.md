# MicroK8S

## Install

### Latest

```text
snap install microk8s --classic
```

### Specific version

```text
snap install microk8s --classic --channel=1.17/stable
```

### Set group

```text
sudo usermod -a -G microk8s $USER
```

Logout from your workstation session and login again.

## Useful commands

### Get kubeconfig

```text
microk8s.config > $HOME/.kube/config
```

### Reset cluster

```text
microk8s reset --destroy-storage
```

### References

[https://microk8s.io/docs/commands](https://microk8s.io/docs/commands)

## Registry

### Push

Pushing to this insecure registry may fail in some versions of Docker unless the daemon is explicitly configured to trust this registry. To address this we need to edit /etc/docker/daemon.json and add:

{% code title="/etc/docker/daemon.json" %}
```text
{
  "insecure-registries" : ["localhost:32000"]
}
```
{% endcode %}

Then restart docker.

```text
sudo systemctl restart docker
```

Enable registry, build and push image.

```text
microk8s enable registry #20Gi registry
#microk8s enable registry:size=40Gi

# Build image
docker build . -t localhost:32000/myimage:registry

# Or tag an existing image
#docker tag 1fe3d8f47868 localhost:32000/myimage:registry

docker push localhost:32000/myimage:registry
```

Deploy it.

```text
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  namespace: myapp
  labels:
    app: myapp
spec:
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: localhost:32000/myimage:registry
        imagePullPolicy: Always
        ports:
        - containerPort: 8000
```

### Remove

Run on your workstation:

```bash
registry=localhost:32000
repositories=$(curl ${registry}/v2/_catalog)
for repo in $(echo "${repositories}" | jq -r '.repositories[]'); do
  echo $repo
  tags=$(curl -sSL "http://${registry}/v2/${repo}/tags/list" | jq -r '.tags[]')
  for tag in $tags; do
    echo $tag
    curl -v -sSL -X DELETE "http://${registry}/v2/${repo}/manifests/$(
      curl -sSL -I \
          -H "Accept: application/vnd.docker.distribution.manifest.v2+json" \
          "http://${registry}/v2/${repo}/manifests/$tag" \
      | awk '$1 == "Docker-Content-Digest:" { print $2 }' \
      | tr -d $'\r' \
    )"
  done
done
```

Then run:

```bash
registry_pod=$(kubectl --namespace="container-registry" get pods --template '{{range .items}}{{.metadata.name}}{{"\n"}}{{end}}')
kubectl exec --namespace="container-registry" $registry_pod /bin/registry garbage-collect /etc/docker/registry/config.yml
```


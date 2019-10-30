# Executing script on K8s Nodes

```bash
#!/bin/sh

# This script will be called in privileged mode on each node of the k8s cluster (with the help of a DeamonSet)
# This is where you should configure the node if you want to (for example change inotify values).
# Nothing guarantees that this script will not run multiple times, so please be sure to make the script idempotent.

# Gracefully handle the TERM signal sent when deleting the daemonset
trap 'exit' TERM

set -ex

# Configure inotify to be able to run a lot a containers on each node.
# By default each node can run a maximum of 128 files.
echo 524288 > /proc/sys/fs/inotify/max_user_instances
echo 524288 > /proc/sys/fs/inotify/max_user_watches

# Prevent the container from exiting and k8s restarting the daemonset pods
while true; do sleep 1; done

```

```text
FROM alpine

COPY configure-node.sh configure-node.sh

CMD ["/bin/sh", "configure-node.sh"]
```

```yaml
kind: DaemonSet
apiVersion: extensions/v1beta1
metadata:
  name: startup-script
  labels:
    app: startup-script
spec:
  template:
    metadata:
      labels:
        app: startup-script
    spec:
      hostPID: true
      containers:
        - name: startup-script
          image: YOUR_DOCKER_IMAGE
          securityContext:
            privileged: true
```


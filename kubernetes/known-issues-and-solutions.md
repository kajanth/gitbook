# Known Issues and solutions

## Nginx ingress

### 308 redirect to HTTPS

#### Solution

Edit ingress configmap.

```text
kubectl -n ingress-external edit configmaps ingress-controller-leader-nginx
```

Add the following annotations:

```text
use-forwarded-headers: "true"
force-ssl-redirect: "false"
```

Restart controller.

```text
kubectl -n ingress-external scale deployment --replicas=0 ingress-external-nginx-ingress-controller
kubectl -n ingress-external scale deployment --replicas=2 ingress-external-nginx-ingress-controller
```

#### References

[https://github.com/kubernetes/ingress-nginx/issues/1957](https://github.com/kubernetes/ingress-nginx/issues/1957)

### Duplicate location "/healthz"

Complete nginx pod log:

```text
2020/01/27 17:12:12 [emerg] 105#105: duplicate location "/healthz" in /tmp/nginx-cfg568474076:487
nginx: [emerg] duplicate location "/healthz" in /tmp/nginx-cfg568474076:487
nginx: configuration file /tmp/nginx-cfg568474076 test failed
```

It happens when you have an ingress object conflicting with "/healthz" path.

#### Solution

Make sure to not have an ingress object overlapping "/healthz".

## Pod sandbox changed, it will be killed and re-created

### Cause

This error happens when deploying a pod. It is caused most liked because of Docker processes crashed or is unstable on the node due IO peak.

### Solution

The solution is to reboot the node.

### References

[https://plugaru.org/2018/05/21/pod-sandbox-changed-it-will-be-killed-and-re-created/](https://plugaru.org/2018/05/21/pod-sandbox-changed-it-will-be-killed-and-re-created/)

## failed to watch file "/var/log/pods/6438eb52-202a-11ea-8dce-e279cb2777e2/my-app/0.log": no space left on device

### Symptom

When you run.

```text
kubectl -n my-ns logs -f my-app-659858b967-5hmtz
```

The command outputs a few lines of log and then breaks.

### Cause

The obvious reason is the node's HD is full. Although this error can be caused by other reasons.

This error \(ENOSPC\) comes from the inotify\_add\_watch syscall, and actually has multiple meanings \(the message comes from golang\). Most likely the problem is from exceeding the maximum number of watches, not filling the disk. This can be increased with the fs.inotify.max\_user\_watches sysctl.

### Solution

Increase max\_user\_watches.

```text
cat /proc/sys/fs/inotify/max_user_watches # default is 8192 
sysctl fs.inotify.max_user_watches=1048576 # increase to 1048576
```

If you do not have SSH connection to the node, apply the following manifest \(not recommended for production environments\).

```text
apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
  name: more-fs-watchers
  namespace: kube-system
  labels:
    app: more-fs-watchers
spec:
  template:
    metadata:
      labels:
        name: more-fs-watchers
    spec:
      hostNetwork: true
      hostPID: true
      hostIPC: true
      initContainers:
        - command:
            - sh
            - -c
            - sysctl -w fs.inotify.max_user_watches=524288;
          image: alpine:3.6
          imagePullPolicy: IfNotPresent
          name: sysctl
          resources: {}
          securityContext:
            privileged: true
          volumeMounts:
            - name: sys
              mountPath: /sys
      containers:
        - resources:
            requests:
              cpu: 0.01
          image: alpine:3.6
          name: sleepforever
          command: ["tail"]
          args: ["-f", "/dev/null"]
      volumes:
        - name: sys
          hostPath:
            path: /sys
```

### References

[https://github.com/google/cadvisor/issues/1581](https://github.com/google/cadvisor/issues/1581)

[https://github.com/Azure/AKS/issues/772](https://github.com/Azure/AKS/issues/772)


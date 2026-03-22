---
title: "Kubernetes Notes"
date: "2026-01-20"
tags: ["kubernetes", "devops", "backend", "cloud"]
---

A running collection of notes from learning Kubernetes — meant for revision. Refer to the official documentation for deeper reading.

## Manifests

The term *manifest* is a generic term — like a shipping manifest — it specifies the details of a resource. Kubernetes reuses that concept throughout.

## kubectl top nodes

For this to work, you need to install the metrics server. The easiest way is to download the manifest from GitHub and apply it.

## Request Quota

Can be specified per namespace. Kubernetes internally enforces this.

## Probes

Probes are added at the container level.

- **Liveness Probe** — if it fails, kill the container
- **Readiness Probe** — if it fails, the container is not ready (don't put it behind a load balancer serving traffic)

## Watch Command
```bash
watch -n 0.1 kubectl describe ns demo
```

Runs `kubectl describe ns demo` every 0.1 seconds. Really useful for observing changes in real time.

## Quick Pod Commands
```bash
k run demopod --image=nginx
k port-forward demopod 2224:80

# In a new tab
curl localhost:2224
k exec -it demopod -- sh
k cp nginx.conf demopod:etc/nginx/nginx.conf
nginx -s reload
```

## ConfigMaps

ConfigMaps let you persist configuration beyond the lifetime of a container.
```bash
k create configmap heroes --from-file=heroes.txt
```

This creates a ConfigMap and stores the file contents as a string. Multiple data sources inside a ConfigMap are not recommended unless they are env variables.

### Mounting a ConfigMap
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: demo-pod
  namespace: demo
spec:
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - name: dc-heroes
      mountPath: /heroes
  volumes:
  - name: dc-heroes
    configMap:
      name: dem-heroes
```

### Using subPath

If you want to mount into a directory that already has other files, use `subPath` so they co-exist:
```yaml
volumeMounts:
- name: dc-heroes
  mountPath: /etc/nginx/heroes.txt
  subPath: heroes.txt
```

## Secrets

Similar to ConfigMaps but for sensitive data like passwords. Important caveat — secrets stored in etcd are **not encrypted** by default. Anyone with access to the API server can read them.
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysql-secret
type: kubernetes.io/basic-auth
stringData:
  password: test123
```

### Using a Secret in a Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mysql-pod
spec:
  containers:
  - image: mysql:8-debian
    name: mysql
    env:
    - name: MYSQL_ROOT_PASSWORD
      valueFrom:
        secretKeyRef:
          name: mysql-secret
          key: password
```

## Logs

Every container inside a pod generates its own logs.
```bash
kubectl logs <pod-name> -c <container-name>
k logs counter -c countby3 --since 10s
k logs counter -c countby3 -f   # continuous/follow
```

Note: logs don't persist after a pod is gone — kubelet cleans them up.

## Labels

Labels are key-value pairs used to group resources.
```bash
k get pods --show-labels
k label pod demo-pod cools=beans
k label pod demo-pod cools-              # remove label
k label pod demo-pod cools=go --overwrite
k get pods --selector=alta3=awesome
k get pods -L app                        # show app label as a column
```

## Deployments vs ReplicaSets

Deployments provision ReplicaSets underneath. The key difference is **rolling deployments** — when a change requires new pods:

1. A new ReplicaSet is created
2. New pods come up in the new ReplicaSet
3. Old pods are gracefully taken down

No downtime.

## Storage

Three important concepts:

**StorageClass** — describes how storage is made (cloud provider, SSD/HDD, performance tier).

**PersistentVolume (PV)** — the actual physical storage unit in the cluster (e.g. EBS volume, NFS share). Exists even if no pod is using it.

**PersistentVolumeClaim (PVC)** — a request for storage. Once it finds a matching PV, it binds to it. One-to-one mapping between PVC and PV.
```
Pod
 ↓ uses
PVC (request)
 ↓ bound to
PV (actual disk)
 ↓ created using
StorageClass (rules)
```

## Services

**ClusterIP** — exposes a stable IP to talk to a set of pods. The IP doesn't change even as pod IPs rotate. Maps `port` on the service to `targetPort` on the container.

### kube-proxy

Every node's kube-proxy knows about all pod IPs across the cluster — they all have identical routing information.

### Service Types

- **ClusterIP** — inter-pod communication within the cluster
- **NodePort** — talk to pods across nodes; publicly accessible via `node-ip:port`. kube-proxy can round-robin to a pod on a different node — completely normal.
- **LoadBalancer** — extension of NodePort; an external technology assigns the IP outside the cluster

## Network Policy

Controls ingress and egress traffic between pods.
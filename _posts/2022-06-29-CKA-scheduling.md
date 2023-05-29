---
title:  "CKA - scheduling"

categories:
  - DevOps
tags:
  - cka
  - devops
  - k8s
# 목차
toc: true
---

### CKA - scheduling

#### 1. Manual Scheduling

> using nodeName 

```yaml
apiVersion : v1
kind : Pod
metadata:
  name: nginx
  labels:
    name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
      - containerPort: 8080
  nodeName: node02
```

#### 2. Selector

```bash
$ kubectl get pods --selector {KEY}={VALUE}
```

#### 3. Taints and Tolerations

> 특정 POD만 스케줄링을 원할 때

> Taints (Node)

```bash
$ kubectl taint nodes {node name} {key}={value}:{NoSchedule | PreferNoSchedule | NoExecute}
```

> Toleration (Pod)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    env: test
spec:
  containers:
  - name: nginx
    image: nginx
  tolerations:
  - key: "example-key"
    operator: "Exists"
    effect: "NoSchedule"
```

#### 3. Node Selectors and Node Affinity

```bash
$ kubectl label nodes {node name} {key}={value}
```

> Node Selector

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    env: test
spec:
  containers:
  - name: nginx
    image: nginx
  nodeSelector:
    {key}: {value}
```

> Node Affinity

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    env: test
spec:
  containers:
  - name: nginx
    image: nginx
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key : {key}
            operator: Exists
```

#### 4. DaemonSet

> 특정 노드 또는 모든 노드에 항상 실행되어야 할 특정 pod을 관리

> use case : kube-proxy, networking

```bash
$ kubectl get ds
```

#### 5. Static Pods

> kube-api-server 없이 worker node 혼자서 pod을 띄울 경우

> 오직 pod만 가능하며, deployment나 service 등은 안됨

> use case : master node의 pod (apiserver, etcd, controller-manager)

```bash
$ vi /etc/kubernetes/manifests/{pod}.yaml

# path configuration
$ vi /var/lib/kubelet/config.yaml
```

#### 6. Multiple Scheduler

> 2개 이상의 scheduler가 있을 경우, pod마다 scheduler를 지정할 수 있음

```bash
# default scheduler
$ vi /etc/kubernetes/manifests/kube-scheduler.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    env: test
spec:
  containers:
  - name: nginx
    image: nginx
  schedulerName: {scheduler name}
```

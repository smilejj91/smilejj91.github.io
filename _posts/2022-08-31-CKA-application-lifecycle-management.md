---
title:  "CKA - application lifecycle management"

categories:
  - DevOps
tags:
  - cka
  - devops
  - k8s
# 목차
toc: true
---

### CKA - application lifecycle management

> Post에서 사용한 사진은 KODEKLOUD에 저작권이 있습니다.

#### 1. Rolling Updates

> 기업에서 application 배포 시 가장 중요하게 생각해야할 점은 바로 24/7 무중단 배포임

> 업데이트, 롤백 등의 과정에서 서비스의 중단이 일어나게 된다면, 고객들은 서비스 이용에 불편함을 겪기 때문에 기업의 이미지에 손실이 생기기 때문

> k8s는 무중단 배포를 Rolling Update 기능을 제공

> 아래의 deployment yaml 파일의 strategy의 type에 RollingUpdate를 명시하고, maxSurge와 maxUnavailable 값을 이용하여 update간 유지될 pod의 개수를 조절할 수 있음

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-devops
  namespace: production
spec:
  selector:
    matchLabels:
      app: hello-devops
  replicas: 2
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 50%
      maxUnavailable: 50%
  template:
    metadata:
      labels:
        app: hello-devops
    spec:
      containers:
      - name: hello-devops-cont
        image: hello-devops:0.0.1
        imagePullPolicy: Always
        resources:
          requests:
            memory: "1Gi"
            cpu: 0.5
          limits:
            memory: "1Gi"
            cpu: 0.5
        ports:
        - containerPort: 8080
```

> update는 변경된 deployment.yaml 를 apply하면 이루어지고, rollback은 rollout을 이용하여 수행할 수 있음

```bash
$ kubectl apply -f hello-devops-deployment.yaml # update
$ kubectl rollout undo deployment/hello-devops -n production # rollback
$ kubectl rollout status deployment/hello-devops -n production # rollback status
$ kubectl rollout history deployment/hello-devops -n production # rollback history
```

#### 2. Configmaps and Secrets

> configmaps은 key - value 형식의 환경변수를 정의한 object이며, secrets는 configmaps의 암호화된 버전이라고 보면 됨

> confimaps와 같은 경우 key - value 형식으로만 저장하는데, 한계가 있기 때문에 k8s document에서 예시한 바와 같이 file 형태로 존재할 수 있도록 작성할 수 있음

[configmap k8s document](https://kubernetes.io/docs/concepts/configuration/configmap/)

> secrets는 ID/PW와 같이 민감한 정보를 암호화하여 저장할 때 사용하거나, private docker registry를 구축할 때에도 사용하니, 해당 부분은 필요시 k8s document에서 찾아보는 것을 추천

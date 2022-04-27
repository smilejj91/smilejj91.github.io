---
title:  "How to setup nextcloud simply on k8s cluster"

categories:
  - DevOps
tags:
  - nextcloud
  - webdav
  - devops
  - k8s
---

### How to setup nextcloud simply on k8s cluster

#### Nextcloud

> NextCloud는 open source이며, 파일 저장 및 공유 서비스를 포함한 업무 관련 기능 (ex. 일정 관리, 문서 작업)들을 제공하는 솔루션으로 dropbox 및 google drive의 대체재로 많이 사용되고 있음

> Authentication 및 웹 인터페이스를 기본적으로 제공해주고 있어서, 보다 안전하고 편리하게 데이터를 관리할 수 있다는 장점이 있음

#### k8s yaml file

> 아래 yaml 파일에는 Secret을 사용하지 않았지만, Secret을 만들어서 사용하는 것을 권장

```bash
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: tmax-nextcloud-ingress
  namespace: osinfra
spec:
  ingressClassName: nginx
  rules:
  - host: nextcloud.tmaxos.net
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: tmax-nextcloud-service
            port:
              number: 80

---

kind: Service
apiVersion: v1
metadata:
  name: tmax-nextcloud-service
  namespace: osinfra
spec:
  type: NodePort
  selector:
    app: tmax-nextcloud
  ports:
    - port: 80 # Default port for image

---

kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: tmax-nextcloud-pvc
  namespace: osinfra
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 10Gi

---

apiVersion: v1
kind: PersistentVolume
metadata:
  name: tmax-nextcloud-pv
  namespace: osinfra
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteMany
  nfs:
    server: 192.168.2.135
    path: /home/share/tmax-nfs/tmax-nextcloud

---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: tmax-nextcloud
  namespace: osinfra
spec:
  selector:
    matchLabels:
      app: tmax-nextcloud
  replicas: 1
  template:
    metadata:
      labels:
        app: tmax-nextcloud
    spec:
      containers:
      - name: tmax-nextcloud
        image: nextcloud:23.0.3
        resources:
          requests:
            memory: "128Mi"
            cpu: 0.5 
          limits:
            memory: "128Mi"
            cpu: 0.5
        ports:
        - containerPort: 80
        volumeMounts:
        - mountPath: /var/www/html
          name: tmax-nextcloud-volume
          subPath: nextcloud
        env:
        - name: MYSQL_PASSWORD
          value: 1234
        - name: MYSQL_DATABASE
          value: nextcloud
        - name: MYSQL_USER
          value: nextcloud
        - name: MYSQL_HOST
          value: 127.0.0.1
      - name: tmax-nextcloud-db
        image: mariadb
        args: ["--transaction-isolation=READ-COMMITTED", "--binlog-format=ROW"]
        resources:
          requests:
            memory: "128Mi"
            cpu: 0.5
          limits:
            memory: "128Mi"
            cpu: 0.5
        ports:
        - containerPort: 3306
        volumeMounts:
        - mountPath: /var/lib/mysql
          name: tmax-nextcloud-volume
          subPath: db
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: 1234
        - name: MYSQL_PASSWORD
          value: 1234
        - name: MYSQL_DATABASE
          value: nextcloud
        - name: MYSQL_USER
          value: nextcloud
      volumes:
      - name: tmax-nextcloud-volume
        persistentVolumeClaim:
          claimName: tmax-nextcloud-pvc

```

> Secret을 사용할 경우 

```bash
apiVersion: v1
kind: Secret
metadata:
  name: tmax-nextcloud-secret
  namespace: tmaxos-infra
type: Opaque
stringData:
  MYSQL_ROOT_PASSWORD: 1234
  MYSQL_PASSWORD: 1234
  MYSQL_DATABASE: nextcloud
  MYSQL_USER: nextcloud
  MYSQL_HOST: 127.0.0.1
```

> Secret을 사용할 경우 Deployment의 env를 아래와 같이 수정

```bash
...
env
  - name: MYSQL_PASSWORD
    valueFrom:
      secretKeyRef:
        name: tmax-nextcloud-secret
        key: MYSQL_PASSWORD
...
```

#### Access nextcloud through browser

```bash
$ http://{ip address}
# ex) http://localhost
```

#### Create Admin account

> 초기화면에서 계정 생성

#### 번외. WebDAV를 통한 파일 업로드 방법

> 파일-> 설정-> WebDAV Address 확인-> curl command를 이용하여 업로드

```bash
$ curl --user '{ID}:{Password}' -T "{업로드할 파일}" "{WebDAV Address}/{하위폴더}/{파일이름}"
```

> 직접 mount를 해서 파일을 업로드하는 방법은 아래 [Accessing Nextcloud files using WebDAV] link 참고

### link

[Official Docker Document](https://hub.docker.com/_/nextcloud)

[Sample yaml file](https://github.com/smilejj91/k8s-cluster-setting/blob/main/app/tmax-nextcloud/tmax-nextcloud.yaml)

[Accessing Nextcloud files using WebDAV](https://docs.nextcloud.com/server/23/user_manual/en/files/access_webdav.html)

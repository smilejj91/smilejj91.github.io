---
title:  "Docker Swarm"

categories:
  - DevOps
tags:
  - docker
  - devops
  - docker-swarm
# 목차
toc: true
---

# Docker Swarm

## 1. What is Docker Swarm

### Architecture (Node)

![Docker Swarm Node Architecture](/assets/img/docker-swarm-node-architecture.png)

* Manager
  * Manager의 주 역할은 cluster 관리와 서비스들의 스케줄링을 담당 (k8s의 master node라고 생각하면 이해하기 쉬움)
  * 1개의 manager로도 동작이 가능하나, 고가용성을 위해 다수 (홀수 개)의 manager를 두는 것을 권장함
  * 만약 manager node에는 container 실행이 안되도록 방지하고 싶다면, manager node를 active 상태에서 drain 상태로 변경하면 됨
```bash
$ docker node update --availability drain (manager node id)
```

* Worker
  * Worker의 주 역할은 실질적인 container 실행을 담당하며, manager가 담당하는 스케줄링이나 cluster 관리를 수행하지 않음
  * 당연하게도, Manager Node없이 Worker Node만 존재할 수 없으며, 최소 1개 이상의 Manager Node가 존재해야함
  * worker node를 manager node로 격상하거나, 반대로 manager node를 worker node로 격하할 수 있음
```bash
$ docker node promote/demote
```

### Architecture (Service)

![Docker Swarm Service Architecture](/assets/img/docker-swarm-service-architecture.png)

* Service
  * Docker에서 배포 단위가 Container 였다면, Docker Swarm의 기본 배포 단위는 Service임
  * 하나의 Service를 정의할 때, 사용할 docker image와 command, port, replica 수, resource limit, environment 등을 정의할 수 있음

* Task
  * Replica 숫자만큼 Task가 생성되며, 이름은 (서비스명).(일련번호) 형식으로 생성됨
  * Service와 Task를 한 줄로 정리하자면, Service는 명세이고 Task는 실질적으로 배포되는 컨테이너임

### Architecture (Stack)

![Docker Swarm Stack Architecture](/assets/img/docker-swarm-stack-architecture.png)

* Stack
  * 하나 이상의 서비스(Service)로 구성된 다중 컨테이너 애플리케이션 묶음을 의미
  * 흔히 알고 있는 docker compose와 유사하지만, default network 구성 등에 있어서 분명한 차이가 있음
  * stack deploy시, 마지막에 stack name을 넣게 됨

```bash
$ docker stack deploy -c $stack_yaml_file $stack_name
```
  * stack 자체의 의미는 애플리케이션 묶음을 의미하지만, 필요시 stack을 k8s namespace와 같이 사용할 수 있음
  * 실제로 아래 커맨드를 통해서 service의 상세 정보를 확인해보면, stack에 대한 namespace 라벨이 존재하는 것을 확인할 수 있음

```bash
$ docker inspect service $service_name
 
... (생략) ...
"Labels": {
                ... (생략) ...
                "com.docker.stack.namespace": "$stack_name"
          },
```

### Architecture (Security)

![Docker Swarm Security Architecture](/assets/img/docker-swarm-security-architecture.png)

* Security
  * 기본적으로 docker swarm을 구성하게 되면, PKI를 사용하여 cluster 보안을 구축한다.
  * Default docker swarm
  * init을 하게 되면 manager node에 root CA를 비롯한 인증서를 생성하게 된다. (파일 위치 : /var/lib/docker/swarm/certificates)
  * docker swarm init 수행 시 --external-ca 옵션을 이용하여, 다른 CA를 설정할 수도 있음

## 2. How to use Docker Swarm

### Setup Docker Swarm Cluster

* Manager Node

```bash
$ docker swarm init # initialize docker swarm
$ docker swarm join-token worker # create token to join swarm cluster for worker node
```

* Worker Node

```bash
$ docker swarm join --token (token) (manager node) # get command by executing command '$docker swarm join-token worker' in manager node
```

### Deploy Service

* Using command

```bash
$ docker service create --replicas 1 --name hello-world alpine ping docker.command
```

* Using Compose File

```bash
$ vi compose.yaml
 
version: "3.7"
 
services:
  nginx:
    image: nginx:latest
    deploy:
      mode: global
      placement:
        constraints: [node.role == worker]
```

```bash
$ docker stack deploy -c compose.yaml nginx
``` 

### Deploy Service

* docker service : https://docs.docker.com/engine/reference/commandline/service/
* docker swarm : https://docs.docker.com/engine/reference/commandline/swarm/


## 3. Swarmpit

### What is Swarmpit

* Docker Swarm Cluster UI 관리 tool
* Node, Service, Stack 등의 상태 모니터링 및 Service 배포까지 가능

### How to deploy swarmpit
```bash
$ vi swarmpit.yaml # https://github.com/swarmpit/swarmpit/blob/master/docker-compose.yml
 
version: '3.3'
 
services:
  app:
    image: swarmpit/swarmpit:latest
    environment:
      - SWARMPIT_DB=http://db:5984
      - SWARMPIT_INFLUXDB=http://influxdb:8086
    volumes:
      
      - /var/run/docker.sock:/var/run/docker.sock:ro
    ports:
      - 888:8080
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080"]
      interval: 60s
      timeout: 10s
      retries: 3
    networks:
      - net
    deploy:
      resources:
        limits:
          cpus: '0.50'
          memory: 1024M
        reservations:
          cpus: '0.25'
          memory: 512M
      placement:
        constraints:
          - node.role == manager
 
  db:
    image: couchdb:2.3.0
    volumes:
      - db-data:/opt/couchdb/data
    networks:
      - net
    deploy:
      resources:
        limits:
          cpus: '0.30'
          memory: 256M
        reservations:
          cpus: '0.15'
          memory: 128M
 
  influxdb:
    image: influxdb:1.8
    volumes:
      - influx-data:/var/lib/influxdb
    networks:
      - net
    deploy:
      resources:
        limits:
          cpus: '0.60'
          memory: 512M
        reservations:
          cpus: '0.30'
          memory: 128M
 
  agent:
    image: swarmpit/agent:latest
    environment:
      - DOCKER_API_VERSION=1.35
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
    networks:
      - net
    deploy:
      mode: global
      labels:
        swarmpit.agent: 'true'
      resources:
        limits:
          cpus: '0.10'
          memory: 64M
        reservations:
          cpus: '0.05'
          memory: 32M
 
networks:
  net:
    driver: overlay
 
volumes:
  db-data:
    driver: local
  influx-data:
    driver: local
```

```bash
$ docker stack deploy -c swarmpit.yaml swarmpit
```

### Dashboard

![Docker Swarm Dashboard](/assets/img/docker-swarm-dashboard.png)

![Docker Swarm Dashboard2](/assets/img/docker-swarm-dashboard2.png)

![Docker Swarm Dashboard3](/assets/img/docker-swarm-dashboard3.png)


### Git

[Docker Swarm Quick Start](https://github.com/smilejj91/devops-setting/tree/main/docker-swarm)

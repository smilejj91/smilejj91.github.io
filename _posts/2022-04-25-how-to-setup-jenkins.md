---
title:  "How to setup jenkins simply using docker-compose"

categories:
  - DevOps
tags:
  - jenkins
  - devops
  - docker
  - docker-compose
# 목차
toc: true
toc_sticky: true
---

# How to setup jenkins using docker

## 1. Jenkins

* An open source automation server which enables developers around the world to reliably build, test, and deploy their software. 

## 2. docker-compose file

* jenkins 폴더를 host에 두어, container 안에서 일어나는 작업물에 대해서 소실되지 않도록 관리

```bash
version: '3.2'
# web container
services:
  jenkins:
    image: jenkins/jenkins:lts
    container_name: jenkins-cont
    user: root # jenkins's account
    ports:
      - "8080:8080" # jenkins port
    volumes:
      - type : "bind"
        source : "${PWD}/jenkins"
        target : "/var/jenkins_home"
```

* user를 root로 설정하지 않을 경우 jenkins의 소유권한을 변경해야함

```bash
$ chown -R jenkins:jenkins ./jenkins
```

## 3. execute docker container

```bash
$ docker-compose up -d
```

## 4. Access jenkins through browser

```bash
$ http://{ip address}:{port}
# ex) http://localhost:8080
```

## 5. Install Jenkins

* insert jenkins/secrets/initialAdminPassword
* install recommend plugin
* make admin account

## 6. link

[Official Document](https://www.jenkins.io/doc/book/installing/docker/)


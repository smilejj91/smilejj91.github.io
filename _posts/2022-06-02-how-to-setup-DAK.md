---
title:  "How to setup DAK"

categories:
  - DevOps
tags:
  - dak
  - linux
  - debian
  - bullseye
  - os
  - devops
# 목차
toc: true
---

# How to setup DAK

## 1. What is DAK (Debian Archive Kit)

* DAK이란 Debian 진영에서 사용하는 레포 관리를 위한 파이썬 스크립트, 쉘 스크립트의 모음과 레포 정보 저장을 위해 구성된 Postgresql DB를 이르는 말

## 2. docker-compose file

```bash
version: '3.2'

services:
  dak:
    image: dak:1.0.1
    container_name: dak-cont
    hostname: ftp-master
    volumes:
      - .:/home/dak/dak
      - /srv/dak-data:/var/lib/postgresql/data
      - /srv/dak:/srv/dak
  uploadqueue:
    image: uploadqueue:1.0.1
    container_name: uploadqueue-cont
    ports:
      - "21:21" # for ftp upload
      - "2022:22" # for dak rsync
    volumes:
      - type: "bind"
        source : "/srv/upload.example.net"
        target : "/srv/upload.example.net"
    links:
      - dak
```

* [dak Dockerfile](https://github.com/smilejj91/dak-setting/blob/master/Dockerfile)
* [uploadqueue Dockerfile](https://github.com/smilejj91/dak-setting/blob/master/tools/debianqueued-0.9/Dockerfile)

## 3. execute docker container

```bash
$ docker-comopse up -d
```

## 4. create new suite

```bash
$ docker exec -it dak-cont bash
$ su - dak
$ cat /home/dak/create_new_suite.sh
```

## 5. uploadqueue: import Maintainer pulic key 

```bash
$ docker exec -it uploadqueue-cont bash
$ su - dak
$ gpg --no-default-keyring --keyring /srv/updoad.example.net/keyrings/upload-keyring.gpg --import {public key}
```

## 6. Test : push packages to DAK

### 6-1. Maintainer : Create GPG key

```bash
$ gpg --gen-key --default-new-key-algo=rsa4096/cert,sign+rsa4096/encr
$ gpg -a --export {gpg name} > {gpg name}.pub
```

* send {gpg name}.pub to DAK administrator

### 6-2. Maintainer : push packages using dput

* dput config setting

```bash
$ apt install dput
$ vi /etc/dput.cf

[ftp-master]
fqdn      = uploadqueue.example.net # register domain name in /etc/hosts
incoming    = /pub/UploadQueue/
login     = anonymous
allow_dcut    = 1
method      = ftp
```

* send signed source packages using dput
  * 참고 : [how to make debian package](https://smilejj91.github.io/os/how-to-make-debian-package/)

```bash
$ sbuild -A -d tmax-unstable --source-only-changes
$ debsign -e "{gpg name}" {source package}_{version}_source.changes
$ dput {source package}_{version}_source.changes
```

## 7. link

[DAK setting](https://github.com/smilejj91/dak-setting)

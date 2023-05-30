---
title:  "Trouble shooting DAK - 1"

categories:
  - DevOps
tags:
  - dak
  - troubleshooting
  - devops
# 목차
toc: true
toc_sticky: true
---

# Trouble shooting DAK

## 1. Uploadqueue vsftpd 오동작 수정

* Uploadqueue docker image를 제작 완료하여 테스트를 하던 도중 uploadqueue로의 패키지 업로드가 정상적으로 되지 않는 문제를 발견
* Uploadqueue에서는 파일 전송을 받기 위해서 vsftpd를 사용하고 있는데, vsftpd가 container가 아닌 host에서 동작할 때는 문제가 발생하지 않는 것으로 보아 container안에서 동작할 때에 필요한 네트워크 설정이 있는 것으로 보임
* 문제의 원인은 vsftpd가 container 내부에서 동작하는 경우, FPT server와 FTP client의 handshake 과정에서 FTP client가 파일을 전송하기 위해 FTP server로부터 받는 host의 IP의 정보가 container namespace에서 사용하는 IP이기 때문
* docker container는 기본적으로 bridge를 사용하여 host와 container의 network namespace를 분리
* 이에 따라, container안에서 동작하는 vsftpd가 host의 IP를 전달할 수 있도록 host와 container의 network namespace를 같이 사용하도록 조치

```bash
$ cat docker-compose.yml
...
network_mode: "host"
...
```

* [DAK docker-compose.yml](https://github.com/smilejj91/dak-setting/blob/master/docker-compose.yml)


## 2. DAK docker image 오류 수정

* DAK을 docker image로 구동할 경우, hostname에 영향을 받는 부분이 있음을 확인
* hostname과config/dak/dak.conf의 Generate-Index-Diffs에 명시되어있는 Archive 값이 같아야 정상적으로 동작하는 것을 확인
* 현재 사내에서 운영중인 dak는 hostname이 ftp-master로 설정이 되어있어야 동작하도록 되어있으므로, hostname을 ftp-master로 설정하도록 docker-compose 옵션 추가
* 상세: dak에서는 패키지들의 directory path (ex. incoming package, release package, etc.)를 archive라는 db table로 관리하고 있으며, 그 중 release package에 대한 record 이름은 container의 hostname으로 설정하도록 되어있음

```bash
$ cat docker-compose.yml
...
hostname: ftp-master
...
```

* [DAK docker-compose.yml](https://github.com/smilejj91/dak-setting/blob/master/docker-compose.yml)

## 3. DAK directory 추가 생성

* 일부 directory가 setup 과정에서 생성되지 않아서 정상적으로 스크립트가 동작하지 않아 directory 수동으로 생성

```bash
$ mkdir -p /srv/dak/queue/byhand/COMMENTS (role : byhand package comment directory)
$ mkdir -p /srv/dak/tiffani (role : temp directory)
$ mkdir -p /srv/dak/ftp/indices/files/components (role : indexing directory)
$ mkdir -p /srv/dak/mirror/ftp-master (role : mirror directory)
```

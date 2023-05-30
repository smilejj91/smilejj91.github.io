---
title:  "What is VSFTPD"

categories:
  - DevOps
tags:
  - vsftpd
  - linux
  - ftp
  - devops
# 목차
toc: true
toc_sticky: true
---

# What is vsftpd

## 1. VSFTP (Very Secure File Transfer Protocol)

* 보안 부분을 강조한 FTP daemon으로 Redhat, Suse, Open-BSD에서 기본 FTP로 채택하고 있음

## 2. FTP 동작 모드

* FTP 동작모드는 Active mode와 Passive mode로 나뉘며, FTP server는 command port (default: 21)와 data port (default : 20)가 나누어져 있음

### 2-1. Active mode

* 서버가 클라이언트에 접속을 하는 방식

![VSFTPD Active mode](/assets/img/vsftpd-active.png)

  * 1. Client는 server의 21번 포트로 접속한 후에 자신이 사용할 두 번째 포트 (= 5151)를 서버에 미리 알려줍니다.
  * 2. Server는 client의 요청에 응답합니다. (acks)
  * 3. Server의 20번 데이터 포트는 Client가 알려준 두 번째 포트로의 접속을 시도합니다.
  * 4. Client가 server의 요청에 응답합니다. (acks)

* Client가 보내준 정보를 기준으로 Server에서 Client의 Data port에 접속을 시도한 후, Client의 요청에 따라 데이터를 전송하는 방식
* 이 방식을 사용할 경우 주의할 점은 IP 공유기 등 사설 IP에서 접속을 시도할 경우, Client의 Data port가 막힐 가능성이 있음 (= 500 illegal port command)
* 즉, FTP server가 Active 모드로 설정이 되어있다면, Client 쪽에서 data session에 연결될 port number를 방화벽에서 풀어주어야 함

### 2-2. Passive mode

* Client가 Server에 접속을 하는 방식

![VSFTPD Passive mode](/assets/img/vsftpd-passive.png)

  * 1. Client가 command port로 접속을 시도합니다. (Passive 모드 연결)
  * 2. Server에서는 사용할 두 번째 포트를 Client에게 알려줍니다.
  * 3. Client는 다른 포트를 열어 server가 알려준 포트로 접속을 시도합니다.
  * 4. Server가 client의 요청에 응답합니다. (acks)

* Data port와 command port 모두 client에서 server로 연결하는 방식
* 즉, Client의 공유기의 간섭없이 server와의 통신이 가능하며, 물론 Server 쪽에서 사용하고자 하는 port를 방화벽에서 풀어주어야 함
* FTP 서버가 Passive mode로 설정되어있을 경우, server측에서 사용될 FTP port 범위를 지정할 수 있음 (pasv_min_port, pasv_max_port)
* [vsftpd configuration](https://web.mit.edu/rhel-doc/5/RHEL-5-manual/Deployment_Guide-en-US/s1-ftp-vsftpd-conf.html)

## 3. 참고

* 현재 dak의 uploadqueue에서 ftp 통신으로 패키지를 받기 위해 사용하고 있음
* uploadqueue를 docker container로 띄우는 과정에서 ftp 통신이 안되는 문제가 있었음
* 해당 문제의 원인은 vsftpd가 container 내부에서 동작하는 경우, FPT server와 FTP client의 handshake 과정에서 FTP client가 파일을 전송하기 위해 FTP server로부터 받는 host의 IP의 정보가 container namespace에서 사용하는 IP이기 때문
* 이에 따라, container안에서 동작하는 vsftpd가 host의 IP를 전달할 수 있도록 host와 container의 network namespace를 같이 사용하도록 조치

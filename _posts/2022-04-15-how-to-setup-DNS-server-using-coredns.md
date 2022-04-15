---
title:  "How to setup DNS sever using coreDNS"

categories:
  - DevOps
tags:
  - k8s
  - devops
  - coreDNS
---

### DNS (Domain Name System)

> 호스트의 도메인 이름을 호스트의 네트워크 주소로 바꾸거나 그 반대의 변환을 수행할

### server setup

> Lightweight하고 containter-based의 DNS server인 Coredns를 사용 (CKA를 준비하면서 알게된 DNS 서버 솔루션임)

#### docker-compose.yml

```yaml
version: '3.2'
services:
    coredns:
        image: coredns/coredns
        container_name: test-coredns
        restart: always # other option: always - if you want persistent through host reboots
        expose:
            - "53"
            - "53/udp"
        ports:
            - "53:53"
            - "53:53/udp"
        volumes:
            - type : "bind"
              source : "${PWD}/config"
              target : "/etc/coredns"
        entrypoint:
            - "/coredns"
            - "-dns.port=53"
            - "-conf=/etc/coredns/Corefile"
```

#### config/Corefile (using hosts plugin)

```bash
example.net {
  hosts {
    192.168.10.10 test.example.net
    ttl 300
    fallthrough
  }
  reload 5s
  whoami
  log
  errors
}

. {
  forward . 8.8.8.8 {
    except example.net
  }
  log
  errors {
        consolidate 5m ".* i/o timeout$" warning
        consolidate 30s "^Failed to .+"
    }
}
```

#### config/Corefile (using file plugin)

```bash
$ cat /etc/coredns/Corefile
tmaxos.net {
  file /etc/coredns/wildcard.tmaxos.net
}
... 생략 ...
```

```bash
$ cat wildcard.tmaxos.net
@                  3600 SOA   127.0.0.1. (
                              127.0.0.1.                 ; address of responsible party
                              2022040801                 ; serial number
                              3600                       ; refresh period
                              600                        ; retry period
                              604800                     ; expire time
                              1800                       ; minimum ttl
                              )
... 생략 ...
gitlab         IN    A 192.168.10.10
*.pages        IN    A 192.168.10.10
```

#### exec command

```bash
$ docker-compose up -d
```

#### change nameserver

> 네트워크 설정에 들어가서, 해당 dns container를 띄운 ip 주소를 nameserver에 입력하면 끝

### Git

[coreDNS-setting](https://github.com/smilejj91/coreDNS-setting)


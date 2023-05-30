---
title:  "CKA - basic concept"

categories:
  - DevOps
tags:
  - cka
  - devops
  - k8s
# 목차
toc: true
toc_sticky: true
---

# CKA - basic concept

* Post에서 사용한 사진은 KODEKLOUD에 저작권이 있습니다.

## 1. Architecture

![K8S Architecture](/assets/img/k8s-architecture.png)

## 2. ETCD

* 간단하게 말하면, Key - Value 저장소
* k8s에서는 Node, Pod, Config, Secret, Account, Role, Binding 등에 대한 Data를 저장함
* ETCD는 1개 이상의 Cluster로 동작할 수 있음 

## 3. kube-apiserver

![K8S kube-apiserver](/assets/img/k8s-kube-apiserver.png)

* Master와 Worker node 사이에서의 교두보 역할
* 유저 인증, Request validation check, data 전달, etcd 업데이트, 스케줄러와 kubelet과 통신 등의 역할 수행

```bash
$ cat /etc/kubernetes/manifests/kube-apiserver.yaml
$ cat /etc/systemd/system/kube-apiserver.service
```

## 4. kube-controller-manager

![K8S kube-controller-manager](/assets/img/k8s-kube-controller-manager.png)

* POD, Node, Deployment 등의 상태를 monitoring 하고, 문제가 생길 시 복원해주는 역할을 수행
* kube-controller-manager는 여러개의 controller로 구성되어있음

```bash
$ cat /etc/kubernetes/manifests/kube-controller-manager.yaml
```

## 5. kube-scheduler

![K8S kube-scheduler](/assets/img/k8s-kube-scheduler.png)

* taints/toleration, node selector, resource limit 등을 고려하여 스케줄링해주는 역할

```bash
$ cat /etc/kubernetes/manifests/kube-scheduler.yaml
```

## 6. kubelet

![K8S kubelet](/assets/img/k8s-kubelet.png)

* Worker node의 선장을 의미하며, node 등록, pod 생성, node와 pod의 monitoring을 담당


## 7. kube-proxy

![K8S kube-proxy](/assets/img/k8s-kube-proxy.png)

* Master와 Worker node에 각각 1개씩 존재하며, Pod network에 대한 정보를 관리하는 역할을 수행




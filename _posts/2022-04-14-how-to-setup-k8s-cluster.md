---
title:  "How to setup on-premise k8s cluster"

categories:
  - DevOps
tags:
  - k8s
  - devops
---

##. How to setup on-premise k8s cluster

###. What is k8s?

* Kubernetes, also known as K8s, is an open-source system for automating deployment, scaling, and management of containerized applications.

###. setup k8s cluster

####. step 1. node setting (debian 기준)

* (mandatory) swapoff, load br_netfilter, install docker, install kublet/kubeadm/kubectl, install systemd-resolved
* (option) install nfs client (using NFS PersistentVolume)
```
#!/bin/bash

swapoff -a

modprobe br_netfilter

cat <<EOF | tee /etc/modules-load.d/k8s.conf 
br_netfilter
EOF

cat <<EOF | tee /etc/sysctl.d/k8s.conf 
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF

sysctl --system

# install docker 
apt-get update
apt-get install -y ca-certificates curl gnupg lsb-release

curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

apt-get update
apt-get install -y docker-ce docker-ce-cli containerd.io
cat <<EOF | sudo tee /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF

systemctl daemon-reload
systemctl restart docker

apt-get update
apt-get install -y apt-transport-https ca-certificates curl
curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
apt-get update
apt-get install -y kubelet kubeadm kubectl
apt-mark hold kubelet kubeadm kubectl

apt-get install -y bash-completion

# kubectl completion on bash-completion dir
kubectl completion bash >/etc/bash_completion.d/kubectl

 # alias kubectl to k 
 echo 'alias k=kubectl' >> ~/.bashrc
 echo 'complete -F __start_kubectl k' >> ~/.bashrc

 apt-get install -y nfs-common
 systemctl start systemd-resolved
```

####. step 2. master node setting (debian 기준)

* kubeadm init with master node host ip
* deploy weave network (you can choose another network solution)
```
#!/bin/bash

set -x
set -e

ADDRESS=$1

if [ -z ${ADDRESS} ]; then
  echo -e "please put argument!"
  exit 1
fi

kubeadm init --pod-network-cidr 10.244.0.0/16 --apiserver-advertise-address=${ADDRESS}

# run regular user
#mkdir -p $HOME/.kube
#sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
#sudo chown $(id -u):$(id -g) $HOME/.kube/config

# run root user
export KUBECONFIG=/etc/kubernetes/admin.conf
echo 'export KUBECONFIG=/etc/kubernetes/admin.conf' >> ~/.bashrc

echo -e "keep token for joining worker node!!"
echo -e "also, you can generate token $ kubeadm token create --print-join-command "

# https://www.weave.works/docs/net/latest/kubernetes/kube-addon/
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
```

####. step 3. slave node setting (debian 기준)

* generate token at master node
```
$ kubeadm token create --print-join-command
```

* run the command created above at slave node
```
$ kubeadm join {master node ip}:6443 --token {token} --discovery-token-ca-cert-hash {token hash}
```

####. step 4. check node status

* check node status at master node
```
$ kubectl get nodes
```

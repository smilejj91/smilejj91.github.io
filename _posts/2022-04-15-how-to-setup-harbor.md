---
title:  "How to setup Harbor"

categories:
  - DevOps
tags:
  - k8s
  - devops
  - docker
  - harbor
---

### What is Harbor?

> Harbor is an open source registry that secures artifacts with policies and role-based access control, ensures images are scanned and free from vulnerabilities, and signs images as trusted. 

> Harbor, a CNCF Graduated project, delivers compliance, performance, and interoperability to help you consistently and securely manage artifacts across cloud native compute platforms like Kubernetes and Docker.

### Quick start to install Harbor

[harbor-setting](https://github.com/smilejj91/devops-setting/tree/main/harbor)

### Install Harbor Step by Step

#### Step 1. Download Harbor

```bash
read -p 'enter the version: ' VERSION

wget https://github.com/goharbor/harbor/releases/download/${VERSION}/harbor-offline-installer-${VERSION}.tgz

tar xvfz harbor-offline-installer-${VERSION}.tgz
```

#### Step 2. Create CA Certificates

```bash
openssl genrsa -out harbor-ca.key 4096
openssl req -x509 -new -nodes -sha512 -days 3650 \
  -subj "/C=KR/O=SMILE/OU=JJ/CN=harbor-ca" \
  -key harbor-ca.key \
  -out harbor-ca.crt
```

#### Step 3. Create Server Certificates

```bash
openssl genrsa -out harbor.example.net.key 4096
openssl req -sha512 -new \
  -subj "/C=KR/O=Tmax/OU=OS1-2/CN=harbor.example.net" \
  -key harbor.example.net.key \
  -out harbor.example.net.csr

cat >v3.ext<<EOF
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names
[alt_names]
DNS.1=harbor.example.net
DNS.2=example
EOF

openssl x509 -req -sha512 -days 3650 \
    -extfile v3.ext \
    -CA harbor-ca.crt -CAkey harbor-ca.key -CAcreateserial \
    -in harbor.example.net.csr \
    -out harbor.example.net.crt

openssl x509 -inform PEM -in harbor.example.net.crt -out harbor.example.net.cert
```

#### Step 4. Copy Certificates file

```bash
mkdir -p /etc/docker/certs.d/harbor.example.net
cp harbor.example.net.cert /etc/docker/certs.d/harbor.example.net/
cp harbor.example.net.key /etc/docker/certs.d/harbor.example.net/
cp harbor-ca.crt /etc/docker/certs.d/harbor.example.net/

cp harbor-ca.crt /usr/local/share/ca-certificates/
cp harbor.example.net.crt /usr/local/share/ca-certificates/
update-ca-certificates
systemctl restart docker
```

#### Step 5. Modify harbor/harbor.yml.tmpl and change the file name

```bash
# modify harbor/harbor.yml.tmpl if you needed
# hostname, port, certificate/prviate_key, harbor_admin_password, database.password, data_volume, etc.

$ cd harbor && cp harbor.yml.tmpl harbor.yml
```

#### Step 6. Make docker-compose yaml file

```bash
$ ./prepare
```

#### Step 7. Start Harbor using docker-compose command

```bash
$ docker-compose up -d
```

### Git

[harbor-setting](https://github.com/smilejj91/devops-setting/tree/main/harbor)


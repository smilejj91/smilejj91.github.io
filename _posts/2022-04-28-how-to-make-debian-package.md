---
title:  "How to make debian package"

categories:
  - OS
tags:
  - linux
  - debian
  - bullseye
  - os
# 목차
toc: true
---

### How to make debian package

#### Step 1. Install Basic Packaging Software

> shell 환경 변수를 setting합니다. 환경변수를 setting하지 않으면 뒤에 나올 dch로 changelog를 만들어 줄 때 user localhost email이 자동으로 들어가기 때문에 수정이 필요.

```bash
$ vi ~/.bashrc
DEBEMAIL="smilejj91@naver.com"
DEBFULLNAME="smilejj91"
export DEBMAIL DEBFULLNAME
```

```bash
$ apt-get install -y build-essential git-buildpackage sbuild schroot debootstrap debhelper tmax-archive-keyring dput piuparts
```

> git config 등록

```bash
$ git config --global user.email "smilejj91@naver.com"
$ git config --global user.name "Jaejeon Lim"
```

#### Step 2. Git Project 생성

> 소스 패키지 다운로드

```bash
$ apt source --download-only {binary package name}
```

```bash
$ gbp import-dsc {source package name}_{version}.dsc
$ cd {source package name}
$ git checkout master
$ git checkout -b devel # branch 정책은 입맛에 맞게
$ git remote add origin {소스 관리를 위해 생성한 git project 주소}
$ git push -u origin --all
$ git push -u origin --tags
```

#### Step 3-1. Fix Package

> debian/source/format 파일에 3.0 (quilt)로 명시된 경우

```bash
$ git checkout devel
$ gbp pq import # patch-queue branch 자동 생성 및 진입

# Fix source 
$ vi {Files}

# Build Test (Optional)
$ debuild -us -uc -b 
$ dpkg -i ../{package}.deb

$ git commit
$ gbp pq export --commit
$ gbp pq drop # patch-queue branch 자동 삭제 및 devel branch 자동 진입
$ gbp dch -c -N {new version} --debian-branch=devel
$ gbp dch -c -R -D {distribution} --debian-branch=devel # change debian/changelog distribution
$ git tag {new version}
$ git push origin devel
```

> debian/source/format 파일에 native로 명시된 경우

```bash
$ git checkout devel

# Fix source
$ vi Files

# Build Test (Optional)
$ debuild -us -uc -b 
$ dpkg -i ../{package}.deb

$ git commit
$ git checkout devel
$ gbp dch -c -N {new version} --debian-branch=devel
$ gbp dch -c -R -D {distribution} --debian-branch=devel # change debian/changelog distribution
$ git tag {new version}
$ git push origin devel
```

#### Step 3-2. Make New Package

> 새로운 패키지를 만드는 경우

```bash
$ mkdir new_package
$ cd new_package
$ git init --initial-branch=debian/master

# 파일 생성 및 수정
## Case 1. Upstream 있는 경우
$ gbp import-orig -u {version} {tarball}
## Case 2. Upstream 없는 경우
$ git add src
$ git commit

# make debian directory
$ apt-get install -y dh-make
$ dh_make -p {package name}_{version} --createorig

# debian/* 파일 적절하게 수정
```

#### Step 4. Sandboxing Build (dist: bullseye, arch: amd64)

> sbuild를 사용하려면 chroot를 생성하고 sbuild 그룹에 자신을 추가

```bash
# 최초 1회 실행
$ sbuild-createchroot --include=eatmydata,ccache,gnupg --arch=amd64 \
--components=main,contrib,non-free bullseye \
/srv/chroot/bullseye-amd64-sbuild http://deb.debian.org/debian
$ sbuild-adduser root
$ cp /usr/share/doc/sbuild/examples/example.sbuildrc ~/.sbuildrc
$ newgrp sbuild

$ sbuild -A -d bullseye --run-piuparts --run-autopkgtest
```

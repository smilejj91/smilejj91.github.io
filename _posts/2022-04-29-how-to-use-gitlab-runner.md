---
title:  "How to use gitlab runner"

categories:
  - DevOps
tags:
  - gitlab
  - gitlab-runner
  - devops
# 목차
toc: true
toc_sticky: true
---

# How to use gitlab runner

## 1. Gitlab-runner

* gitlab-runner는 개발하고 있는 제품의 CI/CD (Continuous Integration / Continuous Deployments)를 가능하게 해주는 Gitlab의 기능 중 하나이다.
*  Gitlab, Group, Project 단위 별로 Gitlab Runner를 등록할 수 있으며, 등록 방법은 다음과 같다.

## 2. Install and Register Gitlab-runner

* Git Project -> Settings -> Ci/CD -> Runners
* 아래는 gitlab에서 가이드하고 있는 gitlab-runner 등록 방법 중 shell 방식을 사용하는 경우임
* gitlab-runner register는 방식이 다양하므로, [Gitlab Runner Register Guide](https://docs.gitlab.com/runner/register/index.html) 를 참고해서 입맛에 맞게 수정한다.

```bash
#!/bin/bash

REGISTER_TOKEN="please-insert-token" # Git Project -> Settings -> Ci/CD -> Runners에서 발급된 token 입력

echo "nameserver 192.168.2.135" > /etc/resolv.conf # gitlab.tmaxos.net을 resolving 하기 위함

curl -L --output /usr/local/bin/gitlab-runner https://gitlab-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-runner-linux-amd64
chmod +x /usr/local/bin/gitlab-runner
useradd --comment 'GitLab Runner' --create-home gitlab-runner --shell /bin/bash
gitlab-runner install --user=gitlab-runner --working-directory=/home/gitlab-runner

gitlab-runner start

gitlab-runner register --non-interactive \
--url http://gitlab.tmaxos.net/ \
--registration-token $REGISTER_TOKEN \
--executor "shell" \
--description "tmaxgooroom sbuild" \
--tag-list "sbuild"

sbuild-adduser gitlab-runner
cp /usr/share/doc/sbuild/examples/example.sbuildrc /home/gitlab-runner/.sbuildrc
```

* gitlab runner를 성공적으로 등록한 뒤에, 해당 project가 gitlab runner에 의해 빌드될 수 있도록 Project 최상단에 .gitlab-ci.yml 작성
* 아래 파일은 debian package의 sbuild를 하기 위한 .gitlab-ci.yml의 sample 파일로써, 자신의 project에 맞게 적절히 수정하여 활용하도록 한다.
  * 참고 사항 : .gitlab-ci.yml 파일이 git project에 남는다는 단점이 있음

```bash
stages:
  - build

build-job:       # This job runs in the build stage, which runs first.
  stage: build
  before_script:
    - git checkout .
    - git clean -xdf
  script:
    - SOURCE=$(cat debian/control | grep ^Source | awk '{print $2}')
    - cd ../
    - apt-get source $SOURCE
    - cd -
    - rm .gitlab-ci.yml # quilt 기반으로 빌드되는 패키지에서는 .gitlab-ci.yml 파일로 인해 sbuild가 실패함
    - sbuild -A -d tmax-unstable --run-piuparts --run-autopkgtest
    - mkdir output
    - mv ../*.deb ../*.dsc ../*.tar.* ../*.build ../*.buildinfo ../*.changes output/
  artifacts:
    paths:
      - output/*
  tags:
    - sbuild
```

* 성공적으로 빌드가 될 경우, Git project -> CI/CD -> Piplelines에서 위에서 명시한 artifacts를 다운로드 받을 수 있음

---
title:  "Trouble shooting nextcloud - 1"

categories:
  - DevOps
tags:
  - nextcloud
  - troubleshooting
  - devops
  - k8s
---

### Trouble shooting nextcloud

#### 1. Nextcloud Background 작업 방식 수정

> AJAX 방식은 page가 load 될 때 작업을 수행하는 방식으로 이는 page loading을 느리게 할 뿐더러, 파일 보존 주기에 따른 자동 삭제 기능도 page가 load 되어야지만 수행되는 문제가 있음

> 이에 따라, nextcloud에서 권장하고 있는 방식은 cron 방식을 이용해, page가 load되지 않아도 주기적으로 background 작업을 수행할 수 있도록 변경

> 설정 -> 기본설정 -> 배경 작업 -> Cron 선택

```bash
<pre>
$ apt update && apt install -y cron vim
$ crontab -u www-data -e
$ service cron restart
</pre>
```

> [Background Jobs Configuration](https://docs.nextcloud.com/server/latest/admin_manual/configuration_server/background_jobs_configuration.html)

#### 2. nextcloud image 버전 downgrade

> 기존에 사용하고 있던 23.0.3 버전의 nextcloud docker image는 내부적으로 php 8 버전을 사용하고 있는데, php 8 버전 이상에서는 deprecated된 기능을 nextcloud에서는 여전히 사용하고 있는 것을 확인

> 데이터를 backup 하는 데 문제는 없으나, 관련 에러 로그가 지속적으로 발생하는 것은 옳지 않아보여, php 7.4 버전을 사용하는 21.0.7 버전의 nextcloud docker image로 교체 진행

> nextcloud는 downgrade를 지원하고 있지 않기 때문에, 재 구축을 진행하였으며 기존의 backup data도 소실되지 않도록 이관 작업을 수행

#### 3. nextcloud 대용량 파일 업로드 config 추가

> 연구소 내에서 운영하고 있는 서비스들 중, 사업 관리 서비스와 같은 경우에는 iso 이미지들을 보관하고 있기 때문에 현재 backup data 용량이 50GB 정도 됨

> 이에 따라, 대용량 파일 업로드 시 nextcloud의 connection time과 upload 가능한 max size를 증가시킬 필요가 있어서, 아래와 같은 config를 추가함

```bash
$ vi /var/ww/html/.user.ini

upload_max_filesize=100G
post_max_size=100G
max_input_time=14400
max_execution_time=14400
memory_limit=512M
```

> [Big File Upload Configuration](https://docs.nextcloud.com/server/latest/admin_manual/configuration_files/big_file_upload_configuration.html)


#### 4. nextcloud trashbin auto clear

> NextCloud가 할당 받은 용량이 가득찰 경우, 데이터 전송이 되지 않는 것을 확인

> 일전에  Files automated tagging plugin과 Retention plugin을 이용하여, 10일이 지난 파일들이 자동으로 삭제가 될 수 있도록 조치를 했었음

> 확인해보니, 삭제된 파일들은 영구적으로 삭제되는 것이 아니라 휴지통으로 이동되어 보관 되고 있었음

> 휴지통에 있는 파일의 보관 주기는 기본으로 30일로 세팅이 되어있기 때문에, 휴지통에 보관된 파일들이 제거되지 않아 새로운 백업파일이 전송되지 못하고 있었음

> 이에 따라, 휴지통에 있는 파일 보관 정책을 변경하여, 최대 1일동안 보관하고 동적으로 공간이 필요할 때 제거가 될 수 있도록 설정을 추가함 (아래와 같이 추가)

```bash
$ vi config/config.php
... 생략 ...
'trashbin_retention_obligation' => 'auto, 1',
... 생략 ...
```

> [Deleted Items Trash Bin](https://docs.nextcloud.com/server/latest/admin_manual/configuration_server/config_sample_php_parameters.html#deleted-items-trash-bin)

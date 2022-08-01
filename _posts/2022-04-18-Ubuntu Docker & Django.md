---
title:  "Ubuntu Docker & Django 프로젝트"
excerpt: ""

# categories:
#   - linux
#   - django
#   - docker
tags:
  - [Docker, Ubuntu, Django]

toc: true
toc_sticky: true
 
date: 2022-04-18
last_modified_at: 2022-04-18

layout: post
summary: django with ubuntu
author: changheum
category: [docker, ubuntu, django]
# thumbnail: /assets/img/posts/code.jpg
keywords: [docker, ubuntu, django]
usemathjax: true
permalink: /blog/django_ubuntu
---

# 우분투 도커 이미지 받기
`docker pull ubuntu`

### "E: Unable to locate package" 에러 
`apt-get update`
`apt update`
`apt install nano`
`apt install nano`

---
# NAS 연결
## NFS 추가 패키지를 설치
`apt-get install nfs-common`

## NAS 마운트
`mount -t nfs 172.27.128.1:/TrafficImage /nas`

---
# Docker 컨테이너 생성/실행
```
docker run -it \
-p 8000:8000 \
-v /watcher:/watcher \
-v /nas:/nas \
--name WatcherDjango ubuntu /bin/bash
```
---

---
title: Docker Cheatsheet
date: 2018-01-12
draft: false
categories:
- 博客
tags:
- docker
- cloud
---

- Delete all images

  > ```for i in `docker images | awk 'NR>1{print $1":"$2}'`;do docker rmi $i;done```

- Aliyun docker accelerator

  > /etc/docker/daemon.json
  > ```{"registry-mirrors": ["https://5s93w9me.mirror.aliyuncs.com"]}```

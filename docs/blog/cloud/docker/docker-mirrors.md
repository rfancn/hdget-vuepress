---
date: '2017-11-01 17:24:40'
draft: false
lastmod: '2017-11-25 20:16:11'
categories:
- 博客
tags:
- cloud
- docker
title: Docker国内镜像设置
---

- [阿里云](#aliyun)
- [Docker中国](#docker-cn)
- [网易镜像中心](https://c.163.com/hub#/m/home/)
- [Daocloud镜像市场](#daocloud)

#### <a name="aliyun"></a>(1) 阿里云
- 方法一：获取阿里云Docker Hub服务专属的加速链接
    - 登录阿里云
    - 跳转到某个image的页面，比如: [CentOS](https://dev.aliyun.com/detail.html?repoId=1198)
    - 镜像地址一行中点击: "点击获取镜像加速器地址", 获取的专属镜像加速链接类似：[系统分配前缀].mirror.aliyuncs.com

- 方法二：参照文档[阿里云Docker镜像加速器](https://yq.aliyun.com/articles/29941)For docker >=1.10

        $ sudo mkdir -p /etc/docker
        $ sudo tee /etc/docker/daemon.json <<-'EOF'
        {
            "registry-mirrors": ["https://5s93w9me.mirror.aliyuncs.com"]
        }
        EOF
        $ sudo systemctl daemon-reload
        $ sudo systemctl restart docker


#### <a name="docker-cn"></a>(2)[Daocloud镜像市场](https://www.docker-cn.com/registry-mirror)
N/A

#### <a name="daocloud"></a>(3)Daocloud镜像市场
- 首先在[Daocloud](http://www.daocloud.io/)进行注册登录。然后点击加速器
- 获取加速器的相关配置

#### (4)一键脚本配置方法
- 下载[docker_mirror.sh](/assets/docker_mirror.sh)
- 执行命令

        # sh docker_mirror.sh <mirror URL>

        阿里云： # sh docker_mirror.sh {your_id}.mirror.aliyuncs.com
        Docker中国: # sh docker_mirror.sh registry.docker-cn.com
        网易：# sh docker_mirror.sh hub-mirror.c.163.com
        ustc：# sh docker_mirror.sh docker.mirrors.ustc.edu.cn
        Daocloud: # sh docker_mirror.sh {your_id}.m.daocloud.io

- 重启docker service
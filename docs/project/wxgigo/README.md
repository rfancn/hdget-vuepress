---
draft: true
lastmod: '2017-11-25 20:16:11'
title: wxgigo
categories:
- 项目
---

#### 参考SDK
wechat client: ltchat, weixinbot
wxmp client: werobot,wechatpy

#### 设计策略
1. 可视化一键部署
2. 基于插件机制的可扩展性
3. 基于容器化的集群方案

#### 标准版
- 单节点etcd/redis
- 单节点rabbitmq或redis
- 无可视化部署，命令行模式
- 有限的插件支持

#### 性能集群版
- 高可靠性
- 自动扩展
- 可视化部署
- 全自动监控
- 高可靠可扩展的配置集群(etcd cluster)
- 高可靠可扩展的消息中间件集群(rabbitmq cluster)
- 高可靠可扩展的wxgigo集群

##### 集群基础技术
[kubernetes](https://kubernetes.io/)

##### RabbitMQ集群
- [Clustered RabbitMQ on Kubernetes](https://www.mirantis.com/blog/clustered-rabbitmq-kubernetes/)
- [RabbitMQ cluster on kubernetes with StatefulSets](https://wesmorgan.svbtle.com/rabbitmq-cluster-on-kubernetes-with-statefulsets)
- [autocluster](https://github.com/rabbitmq/rabbitmq-autocluster)
- [RabbitMQ-Autocluster on K8s StatefulSet Controller](https://github.com/rabbitmq/rabbitmq-autocluster/tree/master/examples/k8s_statefulsets)
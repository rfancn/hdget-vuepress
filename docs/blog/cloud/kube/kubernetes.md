---
date: '2017-11-17 13:34:08'
draft: false
lastmod: '2017-11-25 20:16:11'
categories:
- 博客
tags:
- cloud
- kubernetes
title: kubernetes初体验
---

在开发wxgigo项目时，架构设计为松耦合的消息队列模式，需要多个开源软件配合来实现。怎样跨平台安装以及复杂的部署问题成了急需解决的事情。初步考虑采用docker容器化的解决方案，怎样自动管理容器实现集群化的配置和调度，这个时候kubernetes进入视野。 Kubernetes is an open-source system for automating deployment, scaling, and management of containerized applications. Google招牌为它贴金了，不过真的用起来，问题多多～

下面介绍在virtualbox上面的centos7上面安装测试kubernetes。

#### 第一步 配置安装环境

##### setup_env.sh   
	
```
# Disable swap
swapoff -a
# Disabling SELinux
setenforce 0
# Pass bridged IPv4 traffic to iptables’ chains, This is a requirement for CNI plugins to work
sysctl -w net.bridge.bridge-nf-call-iptables=1
echo "net.bridge.bridge-nf-call-iptables=1" >> /etc/sysctl.conf
# Stop firewall
systemctl stop firewalld && systemctl disable firewalld
```

#### 第二步 安装相应的软件和启动相应服务

##### install_packages.sh

```
yum -y install ebtables ethtool
yum install -y docker
systemctl enable docker && systemctl start docker

cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg
       https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
yum install -y kubelet kubeadm kubectl
systemctl enable kubelet && systemctl start kubelet
```

#### 第三步 配置master node(这里默认选择flannel pod network方案）
```
kubeadm init --pod-network-cidr=10.244.0.0/16
```

#### 第四步 配置work node
```
- 在work node上执行第一二步
- 拷贝第三步输出的token和sha256值到，加入node
# kubeadm join --token xxx <master node>:6443 --discovery-token-ca-cert-hash sha256:yyy
```

#### 常用命令
[kubectl Cheat Sheet](https://kubernetes.io/docs/user-guide/kubectl-cheatsheet/)

- Get all the nodes
```
$ kubectl get nodes -o jsonpath="{.items[*].metadata.name}"
```

- Get master node
```
$ kubectl get nodes -l node-role.kubernetes.io/master -o jsonpath="{.items[*].metadata.name}"
```

#### 常见问题

- Failed to get system container stats for "/system.slice/docker.service": failed to get cgroup stats for "/system.slice/docker.service": failed to get cgroup stats for "/system.slice/docker.service": failed to get container info for "/system.slice/docker.service": unknown container "/system.slice/docker.service"
```
https://stackoverflow.com/questions/46726216/kubelet-fails-to-get-cgroup-stats-for-docker-and-kubelet-services
```

- Unable to connect to the server: x509: certificate signed by unknown authority (possibly because of "crypto/rsa: verification error" while trying to verify candidate authority certificate "kubernetes")

```
Verify that the $HOME/.kube/config file contains a valid certificate, and regenerate a certificate if necessary. 
Another workaround is to overwrite the default kubeconfig for the “admin” user:
mv  $HOME/.kube $HOME/.kube.bak
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

- How to set appropriate privilege to access the kubernete resources when RBAC enabled?
Below is an example how to access kubernetes endpoints resource:
https://kubernetes.default.svc.cluster.local:443/api/v1/namespaces/default/endpoints/<endpoint name>
```
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: <service account name>
---
kind: Role
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: endpoint-reader
rules:
- apiGroups: [""]
  resources: ["endpoints"]
  verbs: ["get"]
---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: endpoint-reader
  namespace: wxgigo
subjects:
- kind: ServiceAccount
  name: <service account name>
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: endpoint-reader
```

[在Kubernetes上使用Sateful Set部署RabbitMQ集群](https://blog.frognew.com/2017/09/kubernetes-rabbitmq-stateful-set.html)
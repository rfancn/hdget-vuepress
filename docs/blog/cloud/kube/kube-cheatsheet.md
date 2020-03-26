---
title: Kubernetes Cheatsheet
date: '2018-01-10'
draft: false
categories:
- 博客
tags:
- kubernetes
- cloud
---
- when using flannel network plugin, it prompts for missing portmap

  > Fixed by downloading: https://github.com/containernetworking/plugins/releases/download/v0.6.0/cni-plugins-amd64-v0.6.0.tgz and extract portmap to /opt/cni/bin/

- Obtain node join token

  > $ ```kubeadm token list | awk 'NR==2{print $1}'```

- Get master hostname

  > $ ```kubectl get nodes -l node-role.kubernetes.io/master -o jsonpath="{.items[*].metadata.name}"```

- List nodes

  > $ ```kubectl get nodes -o jsonpath="{.items[*].metadata.name}"```

- Get master node ip

  > $ ```kubectl cluster-info | head -1 | grep -oE "[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+"```

- Get master node port

  > $ ```kubectl cluster-info | head -1 | grep -oE ':[0-9]+' | cut -f2 -d":"```

- Get master node ip and port string

  > $ ```kubectl cluster-info | head -1 | grep -oE "[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+:[0-9]+"```

- Run a specified image

  > $ ```kubectl run tempname --rm -it --image=python:2-alpine3.7 --command -- sh```

- Expose service to outside

  > $ ```kubectl edit svc <service name> -n <namespace>```

  > Add one of the node's public IP to externalIPs
    e,g: externalIPs: [192.168.2.131]

- Execute command for specific pod

  > $ ```kubectl exec <pod name> <command name>```
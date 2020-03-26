---
title: How to setup rabbitmq cluster in kubernetes
date: '2017-11-17 13:34:08'
draft: false
lastmod: '2017-11-25 20:16:11'
categories:
- 博客
tags:
- kubernetes
- rabbitmq
- cloud
---

There are multiple projects aims to implement this,  here we will use the official one.
This post mainly focus on the pitfalls while apply the official autocluster plugin.

#### Preface

Firstly, please read the official rabbitmq cluster document to get a brief understanding on how rabbitmq cluster works.
Here is the URL: [Rabbitmq Clustering Guide](https://www.rabbitmq.com/clustering.html)

Then please refer to autocluster general settings explaination in: [rabbitmq autoclustre plugin General Settings](https://github.com/aweber/rabbitmq-autocluster/wiki/General%20Settings)

#### Get Ready

We need parepare a docker image with autocluster plugin integrated and enabled, we can build such rabbitmq docker image manually use below Dockerfile:

```
FROM alpine:3.6

# Version of RabbitMQ to install
ENV RABBITMQ_VERSION=3.6.14 \
    ERL_EPMD_PORT=4369 \
    AUTOCLUSTER_VERSION=0.10.0 \
    HOME=/var/lib/rabbitmq \
    PATH=/usr/lib/rabbitmq/sbin:$PATH \
    RABBITMQ_LOGS=- \
    RABBITMQ_SASL_LOGS=- \
    RABBITMQ_DIST_PORT=25672 \
    RABBITMQ_SERVER_ERL_ARGS="+K true +A128 +P 1048576 -kernel inet_default_connect_options [{nodelay,true}]" \
    RABBITMQ_MNESIA_DIR=/var/lib/rabbitmq/mnesia \
    RABBITMQ_PID_FILE=/var/lib/rabbitmq/rabbitmq.pid \
    RABBITMQ_PLUGINS_DIR=/usr/lib/rabbitmq/plugins \
    RABBITMQ_PLUGINS_EXPAND_DIR=/var/lib/rabbitmq/plugins \
    LANG=en_US.UTF-8

RUN \
  apk --update add \
    coreutils curl xz "su-exec>=0.2" \
    erlang erlang-asn1 erlang-crypto erlang-eldap erlang-erts erlang-inets erlang-mnesia \
    erlang-os-mon erlang-public-key erlang-sasl erlang-ssl erlang-syntax-tools erlang-xmerl && \
  curl -sL -o /tmp/rabbitmq-server-generic-unix-${RABBITMQ_VERSION}.tar.gz https://www.rabbitmq.com/releases/rabbitmq-server/v${RABBITMQ_VERSION}/rabbitmq-server-generic-unix-${RABBITMQ_VERSION}.tar.xz && \
  cd /usr/lib/ && \
  tar xf /tmp/rabbitmq-server-generic-unix-${RABBITMQ_VERSION}.tar.gz && \
  rm /tmp/rabbitmq-server-generic-unix-${RABBITMQ_VERSION}.tar.gz && \
  mv /usr/lib/rabbitmq_server-${RABBITMQ_VERSION} /usr/lib/rabbitmq && \
  curl -sL -o /usr/lib/rabbitmq/plugins/autocluster-${AUTOCLUSTER_VERSION}.ez https://github.com/rabbitmq/rabbitmq-autocluster/releases/download/${AUTOCLUSTER_VERSION}/autocluster-${AUTOCLUSTER_VERSION}.ez && \
  curl -sL -o /usr/lib/rabbitmq/plugins/rabbitmq_aws-${AUTOCLUSTER_VERSION}.ez https://github.com/rabbitmq/rabbitmq-autocluster/releases/download/${AUTOCLUSTER_VERSION}/rabbitmq_aws-${AUTOCLUSTER_VERSION}.ez && \
  apk --purge del curl tar gzip xz

COPY root/ /

# Fetch the external plugins and setup RabbitMQ
RUN \
  adduser -D -u 1000 -h $HOME rabbitmq rabbitmq && \
  cp /var/lib/rabbitmq/.erlang.cookie /root/ && \
  chown rabbitmq /var/lib/rabbitmq/.erlang.cookie && \
  chmod 0600 /var/lib/rabbitmq/.erlang.cookie /root/.erlang.cookie && \
  chown -R rabbitmq /usr/lib/rabbitmq /var/lib/rabbitmq && sync && \
  /usr/lib/rabbitmq/sbin/rabbitmq-plugins --offline enable \
    rabbitmq_management \
    rabbitmq_consistent_hash_exchange \
    rabbitmq_federation \
    rabbitmq_federation_management \
    rabbitmq_mqtt \
    rabbitmq_shovel \
    rabbitmq_shovel_management \
    rabbitmq_stomp \
    rabbitmq_web_stomp \
    autocluster

VOLUME $HOME
EXPOSE 4369 5671 5672 15672 25672
ENTRYPOINT ["/launch.sh"]
CMD ["rabbitmq-server"]
```

or use the public one, to simpliy the whole process, we choose to use the prebuild public one here: [pivotalrabbitmq/rabbitmq-autocluster](https://hub.docker.com/r/pivotalrabbitmq/rabbitmq-autocluster/)


#### Pitfall

- 403 Forbidden when get rabbitmq service endpoints

    - Symptom

      ```
      =INFO REPORT==== 14-Nov-2017::10:12:30 ===
      autocluster: GET https://kubernetes.default.svc.cluster.local:443/api/v1/namespaces/default/endpoints/rabbitmq

      =INFO REPORT==== 14-Nov-2017::10:12:30 ===
      autocluster: Response: [{ok,{{"HTTP/1.1",403,"Forbidden"},
                                    [{"date","Tue, 14 Nov 2017 10:12:30 GMT"},
                                    {"content-length","93"},
                                    {"content-type","text/plain"},
                                    {"x-content-type-options","nosniff"}],
                                    "User \"system:serviceaccount:default:default\" cannot get endpoints in the namespace \"default\"."}}]
      ```

    - Workaround

      Pleease refer to "examples/k8s_rbac_statefulsets" in: https://github.com/rabbitmq/rabbitmq-autocluster, I created a PR against that issue, detail information can be referred here: [Add RBAC support ](https://github.com/rabbitmq/rabbitmq-autocluster/pull/59)

- Step maybe_cluster failed with failure: inconsistent_cluster

    - Symptom

      ```
      =INFO REPORT==== 15-Nov-2017::15:16:55 ===
      autocluster: Picked node as the preferred choice for joining: 'rabbit@10.244.2.37'
      =INFO REPORT==== 15-Nov-2017::15:16:55 ===
      autocluster: Running step maybe_cluster
      =ERROR REPORT==== 15-Nov-2017::15:16:55 ===
      Node 'rabbit@10.244.2.37' thinks it's clustered with node 'rabbit@10.244.1.40', but 'rabbit@10.244.1.40' disagrees
      =ERROR REPORT==== 15-Nov-2017::15:16:55 ===
      autocluster: Step maybe_cluster failed, will conitnue nevertheless. Failure reason: Failed to cluster with rabbit@10.244.2.37: {inconsistent_cluster,[78,111,100,101,32,39,114,97,98,98,105,116,64,49,48,46,50,52,52,46,50,46,51,55,39,32,116,104,105,110,107,115,32,105,116,39,115,32,99,108,117,115,116,101,114,101,100,32,119,105,116,104,32,110,111,100,101,32,39,114,97,98,98,105,116,64,49,48,46,50,52,52,46,49,46,52,48,39,44,32,98,117,116,32,39,114,97,98,98,105,116,64,49,48,46,50,52,52,46,49,46,52,48,39,32,100,105,115,97,103,114,101,101,115]}.
      ```

    - Workaround

      I created an issue in autocluster github project, normally it caused by the racing problem. Because auto clean logic runs with 60s interval, while crash node removed itself from the cluseter, it will try rejoin soon. If auto clean logic not scheduled to be run on other nodes at that time, other nodes think the crashed node still in the cluster when crash node try to rejoin again. And if AUTOCLUSTER_FAILURE not set to be "stop", it will continue run as standalone mode by ignoring previoius rejoin failure. In that case, the crashed node will nerver ever retry rejoin the cluster, and the kube pod it running in also will not detect the error. Detail information please refer to my last comment in: https://github.com/rabbitmq/rabbitmq-autocluster/issues/62
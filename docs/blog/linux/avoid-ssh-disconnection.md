---
date: '2017-08-16 21:37:38'
draft: false
lastmod: '2017-11-25 20:16:11'
categories:
- 博客
tags:
- linux
title: putty avoid "server unexpectedly closed network connection"
---

When using putty connect to ssh server, it always get connection lost when idle for sometime, and it reports "server unexpectedly closed network connection", there are two ways to workaround this issue:

* From server side, change "/etc/ssh/sshd_confid" as following, it could bring some security issue, so it is not suggested

        ClientAliveInterval 60
        ClientAliveCountMax 999999999

* From client side, e,g: putty

        1. Load putty stored ssh session or create a new ssh session
        2. Go to 'Connection'->'SSH'->'Key'-> 'Max minutes before rekey(0 for nolimit)'
        3. Change it's value from 60 to 2
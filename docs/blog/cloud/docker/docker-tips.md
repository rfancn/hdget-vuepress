---
date: '2017-11-17 13:06:17'
draft: false
lastmod: '2017-11-25 20:16:11'
categories:
- 博客
tags:
- docker
- cloud
title: How to delete container which reported "device or resource busy"
---

PROBLEM
========
When try to delete container, it failed to remove and reportd below error:

    Error response from daemon: driver "overlay" failed to remove root filesystem for c9cbf0b588644f50d1fec80ba11fac493fb336900e174a4ffe521a30637a0542: remove /var/lib/docker/overlay/81dfae6c47d7919e91acb3947d2c2093d2e3549e16099279852796c7251d3026/merged: device or resource busy

WORKAROUND
===========

   1. Find out which process has mounted this busy mountpoint:

        /var/lib/docker/overlay/81dfae6c47d7919e91acb3947d2c2093d2e3549e16099279852796c7251d3026/merged

        $ sudo find /proc -maxdepth 2 -type f -name "mountinfo" -exec grep -nl 81dfae6c47d {} \;
        /proc/23512/mountinfo
        /proc/23513/mountinfo
        /proc/23610/mountinfo
        /proc/23612/mountinfo

    2. Check what's the process according to above PID

        $ ps axf | grep 23512 | grep -v grep
        23512 ?        Ss     0:00 php-fpm: master process (/etc/php-fpm.conf)

    3. Restart the corresponding service

        e,g: In my case, it is php-fpm processes mounted the busy mountpoint, so we need restart it
        $ sudo systemctl restart php-fpm
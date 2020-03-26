---
date: '2017-11-17 13:34:08'
draft: false
lastmod: '2017-11-25 20:16:11'
categories:
- 博客
tags:
- docker
- cloud
title: 'Docker delete container reports error: "device or resource busy"'
---

#### Symptom
When try to delete container, it failed to remove and reportd below error:

    Error response from daemon: driver "overlay" failed to remove root filesystem for
    c9cb...0542: remove /var/lib/docker/overlay/81df...3026/merged: device or resource busy


#### Solution
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
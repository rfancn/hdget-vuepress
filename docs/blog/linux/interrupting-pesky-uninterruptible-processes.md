---
date: '2017-04-13 10:21:07'
draft: false
lastmod: '2017-11-25 20:16:11'
categories:
- 博客
tags:
- linux
title: Interrupting those pesky “uninterruptible” processes
---

Anyone whose dealt with NFS administration has sooner or later had to confront a downed server that results in a hoard of unkillable processes that build up waiting for the NFS server to return. A quick glance at the NFS mount options in mount(8) will tell us we can mount with the "intr" or "soft" options to instruct our client to allow interrupts or to time out in the case of an unresponsive server, but what if we didn't anticipate it? Or what if we chose to use uniterruptible for a reason such as to avoid risks to data integrity if the application is writing over NFS?

Often the advice you'll find in newsgroups is to stop these processes is to simply reboot the machine, but even that can be problematic since a system shutdown can be hampered by lingering dead NFS mounts.

The reason these processes are unkillable is because it's not actually the process that is running, but rather the kernel is interacting with a device or filesystem on the processes behalf. If you've ever tried to mount a damaged floppy or CD and had the mount command block, then you've run into this. In those cases the solution is simply to eject the media; cause the subsystem to fail and the hung process will then error and exit.

In the case of NFS, the subsystem is handled by a kernel thread called rpciod. The quick and dirty way is:

    kill -9 <pid>

on the rpciod kernel thread:

    killall -9 rpciod

Don't worry, it won't actually die, but it will respond to the signal by delivering errors to processes waiting on it. A more elegant approach is to simply cause rpciod to decide it is not worth while to continue its attempts to reach the server. Bringing down your network interface or setting a firewall rule that will reject with "icmp-port-unreachable" will do the trick... just be patient, it can take a few minutes before rpciod gives up, but it will give up, even with hard mounts. Assuming the offending server was 192.168.0.5, inserting this rule on the client should do the trick.

    iptables -I OUTPUT -d 192.168.0.5 -j REJECT </pid>
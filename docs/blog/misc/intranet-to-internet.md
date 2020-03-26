---
date: '2017-04-13 10:21:07'
draft: false
lastmod: '2017-11-25 20:16:11'
categories:
- 博客
tags:
- tips
title: How to open a secure door in intranet for internet
---

Sometimes we want to access intranet resouces from outside internet world, how can we do this? The usual way is using VPN. But if we don't want to set VPN tunnel, is there any other way to access intranet resources? Now we discuss how to implement this in another way, this is not my idea, just borrow from another geek ;) The precondition is your intranet has http proxy and it supports HTTP:CONN method

Here is my test env:

- Home Server:      Internet accessible server which is exposed by DMZ(I use tp-link810N with ADSL connection)
- Intranet Server:  The intranet server locate in office which only can be accessed by VPN

#### Steps

1. Exposed the home server to internet by setting DMZ on router
2. Login to intranet server via VPN
3. Download and compile the transconnect on the intranet serverURL:[transconnect](http://sourceforge.net/projects/transconnect)
	
	INSTALL INSTRUCTION:

		$make
		$make install
		
4. Enable tconn and make it works(By default, it will be installed in ~/.tconn directory):
5. Type the following on the command line will start TransConnect

		$LD_PRELOAD=$HOME/.tconn/tconn.so
		$export LD_PRELOAD<

6. Type the following on command line (Of the same terminal) will stop TransConnect

		$unset LD_PRELOAD

7. Configure TransConnect

	- get your ip address of your http proxy

			$ping <Your company's http proxy hostname>

	- edit ~/.tconn/tconn.conf
	
			proxyserv  *.*.*.*
			proxyport  *
			
	- Download and build autossh utility on office server: [autossh](http://www.harding.motd.ca/autossh/)
		
			$./configure
			$make
			#make install
		
		!!! - The autossh utility can be found in /usr/local/bin/autossh by default
		!!! - it need some development tools installed on this server to support automake & autoconf
	
	- Create a script as below and set execute privilege for it

			#!/bin/bash
			REMOTE_FWD_PORT=8080
			REMOTE_SSH_PORT=22
			PROXY_HOST=&lt;Office Server HTTP Proxy&gt;
			PROXY_PORT=&lt;Office Server HTTP Proxy Port&gt;
			REMOTE_SSH_HOST=&lt;HOME SERVER HOSTNAME&gt;
			REMOTE_SSH_USER=&lt;SSH USER&gt;   &lt;---- it had better to be unprivileged user
			
			export LD_PRELOAD=/home/&lt;USER&gt;/.tconn/tconn.so
			nohup autossh -M 0 -f \
				-N -o "ServerAliveInterval 60" -o "ServerAliveCountMax 3" \
	            -R $REMOTE_FWD_PORT:$PROXY_HOST:$PROXY_PORT \
	            $REMOTE_SSH_USER@$REMOTE_SSH_HOST &
		
		!!! The PROXY_HOST used in this script should not be the same as the one used in transconnect.Otherise, it doesn't works.
	
	- Add this script to /etc/rc.local or similar files that are run at your system startup if you want it running when system up.
	- Test if intranet http proxy forwarding works on 3rd desktop.
		- login to home server
		
				#netstat -ntl | grep <REMOTE_FWD_PORT>
		
		- if yes, then configure your browser to use the proxy and point to that home server 

				<HOME SERVER HOSTNAME>:<REMOTE_FWD_PORT>

#### Security Consideration
- Please considerate add the forward proxy with authentication function
- Please change the sshd config to set GatewayPorts to be 'clientspecified', and make the forward port only license to localhost

e.g:

1. setup the apache as the forward proxy with authentication function
2. use remoteproxy instruction to forward all requests to localhost:port
3. ssh tunnel will only bind to the localhost not public available



#### Reference:
- http://linuxaria.com/howto/permanent-ssh-tunnels-with-autossh?lang=en
- http://www.harding.motd.ca/autossh/
- http://drupal.star.bnl.gov/STAR/comp/sofi/facility-access/ssh-stable-con
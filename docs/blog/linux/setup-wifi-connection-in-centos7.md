---
date: '2017-06-08 11:49:36'
draft: false
lastmod: '2017-11-25 20:16:11'
categories:
- 博客
tags:
- linux
title: Setup wifi connection in terminal in CentOS7
---

1. Make sure NetworkManager supports wifi and manages the wireless device

        #nmcli d
		DEVICE    TYPE      STATE       CONNECTION
		wlp2s0    wifi      unmanaged   --
	    enp9so    ethernet  unavailable	--
	    lo        loopback  unmanaged
    
    !!! If NetworkManager doesn't manage your wireless ethernet controller then you cannot expect it to see wifi networks and connect to them. NetworkManager would normally manage all devices automatically after a fresh boot. You might want to check presence of the wifi package. If you don't have that package installed, you don't have wifi support in NetworkManager.

        # rpm -qa | grep NetworkManager-wifi
        # rpm -ivh NetworkManager-wifi*.rpm
      
2. If you installed NetworkManager-wifi rpm package, it need restart NetworkManager service

	  	# systemctl restart NetworkManager
	  
3. Connect using nmcli, firstly you need view available wifi networks:

        # nmcli dev wifi list
        * SSID          MODE    CHAN    RATE        SIGNAL    BARS    SECURITY
          tplinkxxx     Infra   11      54 Mbit/s   90        ****    WPA1 WPA2
          ...


4. Then choose the wifi network you want to connect, e,g: tplinkxxx

		# nmcli --ask dev wifi connect tplinkxxx
		Password: 
		

Finally, it will create ifcfg-tplink-xxx.cfg in /etc/sysconfig/network-scripts/ dir, and connected to wifi.
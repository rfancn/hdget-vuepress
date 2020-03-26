---
date: '2017-04-13 10:21:07'
draft: false
lastmod: '2017-11-25 20:16:11'
categories:
- 博客
tags:
- linux
title: Linux终端下翻墙教程
---

##### (1) 配置和安装Shadowsocks

1. 安装Shadowsocks

        pip install shadowsocks
        
2. 配置Shadowsocks

		{
		    "server":"Shadowsocks Server",
		    "server_port":443,
		    "local_address": "127.0.0.1",
		    "local_port":1080,
		    "password":"mypassword",
		    "timeout":300,
		    "method":"aes-256-cfb",
		}
		

3. 在local模式下运行Shadowsocks，实际上这时候shadowsocks充当了socks5客户端，用来与远程的Shadowsocks服务通讯

        sslocal -c /etc/shadowsocks.json -d start
        

##### (2) Http协议转Socks5协议

Linux终端下很多软件都只支持http/https协议，而一般翻墙连接的服务器是使用的socks5协议，上面我们在本地运行的也是只支持socks5代理的shadowssock客户端。这时候我们则需要某个软件实现从http到socks5协议的转换，这样linux终端软件就可以正常使用了。 这里推荐[Privoxy](http://www.privoxy.org/)。 

1. 一般主流发行版都会有相对应的包可供下载和安装。这里以CentOS7为例：

         yum install privoxy
     
2. 编辑/etc/privoxy/cofig文件，添加下面一行来指定所有HTTP代理收到的请求将转发到本地shadowsocks客户端

         forward-socks5 / 127.0.0.1:1080 .
         
    !!! 这里1008是本地Shadowsocks客户端的local_port, Privoxy将把所有请求转发到这个端口上
         
##### (3) 配置Linux终端全局http代理或者直接在软件中指定http代理

        $ export http_proxy=127.0.0.1:8118
        $ export https_proxy=127.0.0.1:8118

!!! 这里8118是Privoxy的默认监听端口，HTTP请求会转发到Privoxy 8118端口，然后被转换成Socks5协议转发到Shadowsocks 1080端口，最后本地Shadowsocks客户端将与远程Shadowsocks服务器通讯 
        
        
最后我们看看运行后的结果是怎样的：

    $ wget https://www.dropbox.com/download?plat=lnx.x86
    --2017-04-09 18:30:13--  https://www.dropbox.com/download?plat=lnx.x86
    正在连接 127.0.0.1:8118... 已连接。
    已发出 Proxy 请求，正在等待回应... 302 Found
    位置：https://clientupdates.dropboxstatic.com/dbx-releng/client/dropbox-lnx.x86-23.4.18.tar.gz [跟随至新的 URL]
    --2017-04-09 18:30:14--  https://clientupdates.dropboxstatic.com/dbx-releng/client/dropbox-lnx.x86-23.4.18.tar.gz
    正在连接 127.0.0.1:8118... 已连接。
    已发出 Proxy 请求，正在等待回应... 200 OK
    长度：66089782 (63M) [binary/octet-stream]
    正在保存至: “download?plat=lnx.x86”

    67% [=============================================================>                               ] 44,392,448  90.1KB/s 剩余 4m 39s
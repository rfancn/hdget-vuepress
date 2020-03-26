---
date: '2017-06-08 13:20:33'
draft: false
lastmod: '2017-11-25 20:16:11'
categories:
- 博客
tags:
- shadowsocks
- linux
title: 史上最全Shadowsocks在各个平台上的安装和配置
---

- [Windows上的安装和配置](#windows)
- [安卓上的安装和配置](#android)
- [苹果上的安装和配置](#ios)

#### Windows上的安装和配置

1. Windows客户端下载: [Windows客户端](http://www.hdget.com/asset/ss-win.zip)
2. Windows客户端安装配置

- 安装前请先检查你windows系统的Microsoft.NET Framework 的版本。如果Microsoft.NET Framework的版本低于4.0请升级到4.0以上的版本(含4.0版本),点击控制面板—点击卸载程序—找到Microsoft.NET Framework 查看版本
	
    ![检测Microsoft.NET Framework版本](./check-framework.png#img-fluid)
	
	
    > 如果Microsoft.NET Framework的版本低于4.0，请先安装Microsoft.NET Framework的高版本。
    > [Microsoft.NET Framework下载地址](http://www.microsoft.com/net/download/framework)
	
- 将Windows客户端下载并解压缩到某个合适的目录，双击带纸飞机图标的执行文件Shadowsocks.exe运行软件。输入正确的Shadowsocks服务器地址、端口、密码和选择与服务器匹配的加密方式，点击确定。

    ![填写客户端信息](./config-server.png#img-fluid)

	
- 成功后会在桌面右下方显示Shadowsocks客户端的图标,其显示为一个纸飞机的图标。然后右键点击右下角纸飞机图标打开菜单,选择“启用系统代理”

    ![启动系统代理](./start-proxy.png#img-fluid)


- 选择"系统代理模式"，点击"PAC模式"。

    ![选择pac模式](./choose-proxy-mode.png#img-fluid)

	
	> PAC模式和全局模式的区别在于，PAC模式会根据一些规则判断你所访问的网站是需要走代理还是不需要走代理，也就是所谓的智能切换代理，而全局模式则会将所有访问请求都走代理。 > 例如：github网站在天朝没有被封禁，在PAC模式下判断不会走代理，但有时候访问速度会非常缓慢，这个时候你可以切换到全局模式，强制让访问github的请求走代理以提高访问速度。
	
- 最后，打开浏览器，访问Google搜索或者Youtube，如果可以访问，证明已经可以正常使用。如果出现访问国内网站问题，你可以随时右键点击右下角纸飞机图标，取消“启用系统代理”。 这里强烈建议右键点击右下角纸飞机图标，选择“开机启动”来使得Shadowssock客户端每次可以随机启动。

#### 安卓上的安装和配置

1. 安卓客户端下载: [安卓客户端](http://www.hdget.com/asset/ss-android-latest.apk)
2. 安卓客户端安装和配置

- 首先保证[安卓系统手机开启允许安装未知来源软件功能](http://jingyan.baidu.com/article/03b2f78c1eb0d35ea237ae8d.html)
- 下载并安装安卓客户端后会显示如下界面,请点击红框标识区域，编辑服务器配置
	
    ![软件默认主界面](./android-1.jpg#img-fluid)
	
- 按图示步骤填写服务器的各项参数,并保存

	![编辑服务器配置](./android-2.png#img-fluid)
	
- 保存完后会返回到主界面，按图示启动Shadowsocks服务

	![启动Shadowsocks服务](./android-3.png#img-fluid)
	
- 确认连接

	![确认连接](./android-4.jpg#img-fluid)
	
- 检测并保证连接正常

	![检测并保证连接正常](./android-5.png#img-fluid)
	
#### 苹果iOS上的安装和配置
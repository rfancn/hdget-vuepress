---
date: '2017-06-23 12:02:00'
draft: false
lastmod: '2017-11-25 20:16:11'
categories:
- 博客
tags:
- tips
title: 梅林固件设置
---

最近败了一个Netgear6900，原装Netgear固件运行很稳定，但可玩性不高，可惜了这个双核1G的CPU和256M内存，忍耐不住刷了梅林固件。基于华硕和网件基本采用同一个解决方案开发，使用网件的硬件稍作修改用华硕的固件也可以刷，基于第三方定制开发的华硕固件，叫做梅林(Merlin)。 
- 原始的梅林固件来自于：[A custom firmware for Asus routers](https://asuswrt.lostrealm.ca/)
- 原始的梅林源代码在Github上可以找到：[asuswrt-merlin](https://github.com/RMerl/asuswrt-merlin)
- 国人在梅林固件上的二次开发定制大本营： [Koolshare](http://koolshare.cn/)

因为主要在大陆使用，这篇文章基本以koolshare国产的梅林固件为例，在这里可以下载到[koolshare固件](http://firmware.koolshare.cn/), 网件和华硕的基本位于： merlin_8wan_firmware和merline_xiaobao_firmware目录下。对于同一套硬件解决方案，只是例如少了个USB接口之类的改变的不同网件或者华硕硬件来说，最后的梅林固件都通用知名度最高的型号版本。例如：R6900的使用R7000型号的固件，只是在从官方固件刷到最终梅林固件过程中，中间刷的固件有不同。

刷固件的教程很多，直接在koolshare上可以搜索得到，但具体系统设置以保证稳定或者快速的无线网络不太多，下面我把我测试得到的结果分享一下。

#### 无线网络

无线网络设置比较复杂专业，不同的设备之间的兼容性可能存在比较大的区别，非通讯专业的在设置这部分的时候主要靠不断的测试，我就是起码重启了几十次，测试出了比较符合我的网络设备和工作环境的参数，希望对大家有帮助。参与测试的网络设备有：

- iPhone手机
- 小米电视
- 基于高通解决方案的小米手机
- 华为的安卓手机
- Qualcomm Atheros AR938x无线网卡
- 笔记本自带Intel AC系列无线网卡

以上测试全部基于最新的R7000_380.66_4-X7.5-610梅林固件

1. 关于不同无线地区的设置，测试过全区域、澳大利亚、美国和日本等等，对无线覆盖的范围和信号强度没有太大区别，而且如果设置为美国那些地区，2.4G频段下支持信道最高到11，很多国产无线路由器默认使用了这个频道，也就是说你的邻居很大概率使用这个频道，会对你的无线网络传输造成比较大的干扰。经过十几次重启测试，发现最适合和稳定的还是默认的中国区域。

	![无线网络区域](./wireless-area.png?classes=img-fluid)

2. 关于2.4G无线网络的设置，可以下载一个叫做wifi分析仪的app, 在家里不同地方看有什么wifi信号，强度分别是怎样。理想状态是要设置的信道和该点其他wifi信号不重合，即使保证不了不重合也要尽量少和附近最强信号源的信道交叉。 数值越低的表示信号越强，例如-59db比-70信号强。一般在某点肯定能找到几个信号比较强的wifi源，尽量避开那些加强的，重合也要重合在比较弱的信号源的信道上。 总的来说一般是在1,6,13里面选择，但为了兼容性出发，很多国内的网络配件默认选择11信道，或者1信道。根据不断测试，这里13信道比较适合我的情况。但设置为13信道，AR938x无线网卡找不到2.4G信号，应该是该卡只支持美国标准，毕竟是淘宝货来源复杂，但该网卡在台式机上使用，能连接上5G信号对我来说也没有问题，其他设备能找到并使用就好了。

	![无线网络区域](./wireless-2.4G.png?classes=img-fluid)

3. 关于5G信号，坑比较多，默认选择的149信道很多设备不支持。 如果选择153信道，但有时候能找到能连接少，有时候又不能找到连接。经过不断测试，发现频道带宽设置为40MHZ最佳，其他的频道带宽要不连接不稳定，要不就跑不满带宽。 最终选择153信道，其他比它更高的信道没有测试，有兴趣的可以继续测试。

	![无线网络区域](./wireless-5G.png?classes=img-fluid)


#### 其他玩机参考

- [玩转路由之 AsusWRT-Merlin 与 Entware](https://blog.bluerain.io/p/AsusWRT-Merlin.html)
- [asuswrt-notes](https://github.com/bryfry/asuswrt-notes)
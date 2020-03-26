---
date: '2017-04-13 10:21:06'
draft: false
lastmod: '2017-11-25 20:16:11'
categories:
- 博客
tags:
- tutorial
- webrobot
title: Web抓取分析机器人连载(五)
---

上一篇中我们通过Firebug定位到了想要分析的内容以及其在HTML页面中处于一个什么样的位置。这一篇我们将详细介绍怎样用lxml来解析HTML页面。

### lxml库的安装和配置

本人在Linux机器上开发，Linux操作系统是Oracle Linux 6 update 2, 使用下面最简单的方法安装lxml.

	#easy_install lxml 


其它平台上的安装方法详见: http://lxml.de/installation.html

#### lxml库的使用方法

在本教程中，lxml库主要使用xpath来解析html页面，定位到我们所要关注的html元素，要了解的几个主要lxml的方法如下:

- HTMLElement = lxml.html.parse(filename) 从文件中解析HTML内容
- HTMLElement = lxml.html.fromstring(HTMLString) - 从输入的HTMLString字符串中解析HTML内容 
- HTMLElelment List = <htmlelement>.xpath(Xpathstring) - HTMLElement中执行xpathstring指定xpath语法，找到了xpath语法指定的HTML元素后返回一个解析后的HTMLElement列表，否则返回None

这里有同学问，啥是xpath, 这个不是我们教程关注的内容，你可以查看xpath语法或者教程来了解xpath的使用方法: [xpath_syntax](www.w3school.com.cn/xpath/xpath_syntax.asp), 简单来说你只要知道xpath类似于一种定位某html元素的一个路径信息。 比如某条路径信息定义为获取在HTML页面中所有宽度为710px的表格元素，那么我们就可以用如下语法: ```//table[@width='710']```, 比如我们要获取在当前路径下第一个表格的第一行第一列的内容，我们可以用如下语法：```.//table[1]/tr/td``` ，够简单吧！ 告诉解析器我们要A栋502房的内容，我们就可以获取该房间内的所有东东，不管是家电还是床铺，一条语句搞定。

#### 用xpath获取蓝色、绿色和棕色表格内容

经过我们上一篇对新闻索引页的HTML源代码分析，上一篇中几种不同颜色表格包含不同内容，分别以下面的格式出现：

- 外部蓝色外部大表格都是以这样的格式出现, 那么我们可以xpath语法:```//table[@width='710']```， 这里告诉解析器我们要获取所有属性width为700的table元素。

		<table width="710" border="0" cellspacing="0" cellpadding="0" style="margin-top: 10px; border: 1px solid #9D9C77;">
	
- 包含新闻版块信息的绿色表格都是以这样的格式出现, 那么我们可以用xpath语法```//table[@width='710']/descendant::table[1]```

		</table><table width="100%" border="0" cellspacing="0" cellpadding="0"> 

- 包含新闻标题和链接的棕色表格都是以这样的格式出现, 那么我们可以用xpath语法```//table[@width='710']/descendant::table[2]```

		</table><table width="100%" cellspacing="1" cellpadding="3" border="0"> 


下面我们一些代码做个实验, 首先用Linux上面的wget或者curl工具将新闻索引列表页面抓取下来:

	# wget http://cswb.changsha.cn/html/2012-10/29/index_2012-10-29.htm
	
	--2012-10-29 15:52:04--  http://cswb.changsha.cn/html/2012-10/29/index_2012-10-29.htm
	正在解析主机 cswb.changsha.cn... 113.240.237.125, 222.247.40.20
	正在连接 cswb.changsha.cn|113.240.237.125|:80... 已连接。
	已发出 HTTP 请求，正在等待回应... 200 OK
	长度：271875 (266K) [text/html]
	正在保存至: “index_2012-10-29.htm”
	100%[===================================================================================================================>] 271,875      395K/s   in 0.7s
	2012-10-29 15:52:04 (395 KB/s) - 已保存 “index_2012-10-29.htm” [271875/271875])


进入python交互模式，尝试来解析抓取后的HTML页面内容:

	# python
	Python 2.6.6 (r266:84292, Sep 11 2012, 05:13:12)
	[GCC 4.4.6 20120305 (Red Hat 4.4.6-4)] on linux2
	Type "help", "copyright", "credits" or "license" for more information.
	>>> from lxml import html
	>>> parsed_html = html.parse("index_2012-10-29.htm")
	>>> outside_table_list = parsed_html.xpath("//table[@width='710']")
	>>> len(outside_table_list)
	28
	>>> outside_table_list
	[<element table at 0xb73ff6bc>, </element><element table at 0xb71cc7dc>, </element><element table at 0xb71cc80c>, </element><element table at 0xb71cc83c>, </element><element table at 0xb71cc86c>, </element><element table at 0xb71cc89c>, </element><element table at 0xb71cc8cc>, </element><element table at 0xb71cc8fc>, </element><element table at 0xb71cc92c>, </element><element table at 0xb71cc95c>, </element><element table at 0xb71cc98c>, </element><element table at 0xb71cc9bc>, </element><element table at 0xb71cc9ec>, </element><element table at 0xb71cca1c>, </element><element table at 0xb71cca4c>, </element><element table at 0xb71cca7c>, </element><element table at 0xb71ccaac>, </element><element table at 0xb71ccadc>, </element><element table at 0xb71ccb0c>, </element><element table at 0xb71ccb3c>, </element><element table at 0xb71ccb6c>, </element><element table at 0xb71ccb9c>, </element><element table at 0xb71ccbcc>, </element><element table at 0xb71ccbfc>, </element><element table at 0xb71ccc2c>, </element><element table at 0xb71ccc5c>, </element><element table at 0xb71ccc8c>, </element><element table at 0xb71cccbc>]
	>>>  outside_table = outside_table_list[0]
	>>> outside_table
	</element><element table at 0xb73ff6bc>
	>>> outside_table.tag
	'table'
	>>> outside_table.attrib
	{'cellpadding': '0', 'width': '710', 'style': 'margin-top: 10px; border: 1px solid #9D9C77;', 'cellspacing': '0', 'border': '0'}


从上面代码中可以看到我们获取到的width=710的表格有28个，我们取其中一个，打印出其tag为"table"，attrib为{'cellpadding': '0', 'width': '710', 'style': 'margin-top: 10px; border: 1px solid #9D9C77;', 'cellspacing': '0', 'border': '0'} ， 这证明我们的代码成功获取到了想要得到外部蓝色表格元素。 至于绿色表格，我们可以这样获取：

	>>> green_table_list = parsed_html.xpath("//table[@width='710']/descendant::table[1]")
	>>> green_table = green_table_list[0]
	>>> green_table.tag
	'table'
	>>> green_table.attrib
	{'cellpadding': '0', 'width': '100%', 'cellspacing': '0', 'border': '0'}


同样的方法，我们可以获取棕色表格：

	>>> brown_table_list =  parsed_html.xpath("//table[@width='710']/descendant::table[2]")
	>>> brown_table_list
	[</element><element table at 0xb71dd29c>, </element><element table at 0xb71dd2cc>, </element><element table at 0xb71dd2fc>, </element><element table at 0xb71dd32c>, </element><element table at 0xb71dd35c>, </element><element table at 0xb71dd38c>, </element><element table at 0xb71dd3bc>, </element><element table at 0xb71dd3ec>, </element><element table at 0xb71dd41c>, </element><element table at 0xb71dd44c>, </element><element table at 0xb71dd47c>, </element><element table at 0xb71dd4ac>, </element><element table at 0xb71dd4dc>, </element><element table at 0xb71dd50c>, </element><element table at 0xb71dd53c>, </element><element table at 0xb71dd56c>, </element><element table at 0xb71dd59c>, </element><element table at 0xb71dd5cc>, </element><element table at 0xb71dd5fc>, </element><element table at 0xb71dd62c>, </element><element table at 0xb71dd65c>, </element><element table at 0xb71dd68c>, </element><element table at 0xb71dd6bc>, </element><element table at 0xb71dd6ec>, </element><element table at 0xb71dd71c>, </element><element table at 0xb71dd74c>, </element><element table at 0xb71dd77c>, </element><element table at 0xb71dd7ac>]
	>>> brown_table = brown_table_list[0]
	>>> brown_table.tag
	'table'
	>>> brown_table.attrib
	{'cellpadding': '3', 'width': '100%', 'cellspacing': '1', 'border': '0'}</element></table></htmlelement>
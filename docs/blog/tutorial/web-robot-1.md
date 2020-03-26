---
date: '2017-04-13 10:21:06'
draft: false
lastmod: '2017-11-25 20:16:11'
categories:
- 博客
tags:
- tutorial
- webrobot
title: Web抓取分析机器人连载(一)
---

一般来说，如果某个网站没有提供API来获取其内容，那就只有通过模拟浏览器的行为来抓取其页面来获取我们感兴趣的信息。 理论上来说，只要网站对公共开放，其上面的内容就可以获取，但是如果需要二次验证，比如输入图形验证码或者短信验证码才能查看，这个就稍显复杂，不过也不是没有办法。 本教程关注没有特别保护的页面的抓取，用python来开发，其他语言开发者可以依葫芦画瓢。 本教程虽然关注于内容的自动获取，实际上融汇贯通后完全可以模拟浏览器的操作，年前很热门的12306抢票软件大概也就是此原理开发的。各位看官觉得有用点击下广告也算不费俺一片苦心 :)

#### 开发前环境准备：

- python开发环境
- firefox浏览器
- firefox浏览器安装了httpfox和firebug插件
- libxml2, libxml2-devel, libxslt-devel, libxslt
- [lxml](http://lxml.de)

		#yum install libxml2 libxml2-devel 
		#yum install libxslt libxslt-devel 
		#easy_install lxml 


	!!! 备注：选择lxml作为html的解析器，是因为xpath解析html/xml比较直观也具有较好的容错性，python自带的HTMLParser语法拖沓且容错性不好。我们只要写好xpath语法，基本就可以很直观的获取我们感兴趣的内容。 lxml是在libxml2和libxslt上的一个pythonic的wrapper,能够比较好的支持xpath和xslt.

#### 开发前必备知识：

- http协议基本知识
- python基本知识

#### 在开发前建议先自学下面两篇教程:

- [urllib2 - HOWTO Fetch Internet Resources with Python](http://www.voidspace.org.uk/python/articles/urllib2.shtml)
- [Handling Cookies in Python](http://www.voidspace.org.uk/python/articles/cookielib.shtml)


#### 另外参考：

- [XPath教程](http://www.w3school.com.cn/xpath/index.asp)
- [lxml参考文档](http://lxml.de/lxmlhtml.html)
</ul>

好，一切就绪，马上登场！这里我们以“长沙晚报数字报刊平台“为抓取对象, 具体的抓取URL为： [http://cswb.changsha.cn/html/2012-10/11/index_2012-10-11.htm](http://cswb.changsha.cn/html/2012-10/11/index_2012-10-11.htm)

这是一个典型的SEO友好型的URL，其中随着每天的日期不同这个URL也会不同，上面的链接显示的为2012-10-11号的所有新闻索引，那么我们要抓取的URL每天也不同。例如想获取2012-09-11的新闻，我们就要构造http://cswb.changsha.cn/html/2012-09/11/index_2012-09-11.htm

总的开发步骤为：

1. 构造有效的URL 
2. 模拟浏览器获取URL端的页面 
3. 解析我们感兴趣的页面中的内容
4. 重新组织我们抓取的内容，保存或者重新格式化

今天到此为止，请关注该连载教程下一篇：[Web抓取分析机器人开发教程连载(二)](http://www.guanxigo.com/webrobot-002/)
---
date: '2017-04-13 10:21:06'
draft: false
lastmod: '2017-11-25 20:16:11'
categories:
- 博客
tags:
- tutorial
- webrobot
title: Web抓取分析机器人连载(二)
---

在第一篇中我们确定了几个任务， 在这一篇中我们将完成前面两个任务：

- 构造有效的URL
- 模拟浏览器获取URL端的页面


#### 构造有效的URL

这个任务很容易，利用python的datetime库我们很容易可以合适的日期对象。 注意为了构造合法的URL，这里月和日字符串都是必须符合一定规范的，比如年需要是4个字符的，月和日都是2个字符的，例如：1月必须用01月来表示。 不多罗嗦了，上代码：

	from datetime import date
	
	today = date.today()
	year = when.strftime("%Y")
	month = when.strftime("%m")
	day = when.strftime("%d")
	index_filename = 'index_%s-%s-%s.htm' % (year, month, day)
	index_URL = 'http://cswb.changsha.cn/html/%s-%s/%s/%s' % (year, month, day, index_filename)
	print index_URL


#### 模拟浏览器抓取页面

既然我们在前面已经生成了合法的URL，那么我们的机器人可以大显身手了。如果有同学认真阅读了我第一篇中推荐的关于urllib2和cookielib的教程， 下面的代码应该对你来说很熟悉。

	import urllib2
	import cookielib
	import os
	
	COOKIEFILE='/tmp/csnews.cookie'
	cj = cookielib.LWPCookieJar()


这里我们使用cookielib.LWPCookieJar是因为我想以能够理解的方式查看cookie中的内容，各位可以选择其他的cookiejar存储方式。其实我们抓取对象长沙晚报网站没有返回任何cookie给客户端（浏览器），这里cookie的处理可以忽略，但是作为一个具有完整功能web抓取机器人，这个功能怎么也得备着，江湖险恶多一招防身总是不错的！ 下面的代码实例化并安装具备cookie支持功能的opener作为缺省opener。（这个urllib2的设计方法还是不错的，用了软件设计里面的command模式，一系列的handler可以注册后依次调用，扯远了!)

	if os.path.isfile(COOKIEFILE):
    	cj.load(COOKIEFILE)

	opener = urllib2.build_opener(urllib2.HTTPCookieProcessor(cj))
	urllib2.install_opener(opener)


下面的代码是构造request请求，并抓取页面:

	http_headers = {'User-Agen':"Mozilla/5.0 (Windows NT 6.1; WOW64; rv:15.0) Gecko/20100101 Firefox/15.0.1"}
	req = urllib2.Request(index_URL, None, http_headers)
	try:
	  response = urllib2.urlopen(req)
	except IOError, e:
	  print "We failed to open %s." % index_URL
	  if hasattr(e, 'code'):
	    print "We failed with code - %s" % e.code
	  elif hasattr(e, 'reason'):
	    print "We failed to reach a server."
	    print "This usually means the server doesn't exist/down,or we don't have an internet connection."
	    print "The reason is $s" % e.reason
	else:
	    print "Here are the headers of the page:"
	    print response.info()


http_headers里面我们定义了User-Agent, 有同学问为什么要定义这http头部信息。有这么几个原因：

1. 有的服务器端限制了只允许某些浏览器的访问
2. 有时候服务器端可以根据浏览器的类型来生成合适该浏览器的html页面
3. 最后一点就是为了防止服务器端检测出有自动的机器人来抓取其内容来封杀，嘿嘿，做贼心虚！


未完继续，请关注下一篇： [Web抓取分析机器人开发教程连载(三)](http://www.guanxigo.com/webrobot-003/)
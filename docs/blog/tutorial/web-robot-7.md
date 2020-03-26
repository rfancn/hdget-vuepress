---
date: '2017-04-13 10:21:06'
draft: false
lastmod: '2017-11-25 20:16:11'
categories:
- 博客
tags:
- tutorial
- webrobot
title: Web抓取分析机器人连载(七)
---

啰里啰嗦写了好几篇了，也不知道对大家有没有帮助，这一篇应该是该系列教程的最后一篇了，希望有个好的结尾！上一篇中我们简单介绍了解析新闻索引中的新闻版块，新闻标题和新闻URL的信息，这一篇我们将根据这些信息来逐步获取真正的新闻内容。

首先，我们重新定义一个函数来获取长沙晚报新闻网的带绝对路径的URL地址，总共有两个URL，一个是新闻索引URL，一个是新闻URL. 代码如下：

	def get_absolute_url(type, when, filename = None):
	    if not isinstance(when, date):
	        return None
	    if type == 'news' and filename is None:
	       return None
	    year = when.strftime("%Y")
	    month = when.strftime("%m")
	    day = when.strftime("%d")
	    prefix = 'http://cswb.changsha.cn/html/%s-%s/%s/' %  (year, month, day)
	    if type == 'index':
	        filename = 'index_%s-%s-%s.htm' % (year, month, day)
	    return "%s/%s" % (prefix, filename)


然后我们定义获取web页面的函数如下:

	def fetch_url(url):
	    http_headers = {'User-Agen':"Mozilla/5.0 (Windows NT 6.1; WOW64; rv:15.0) Gecko/20100101 Firefox/15.0.1"}
	    req = urllib2.Request(url, None, http_headers)
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
	            return False
	        else:
	            print "Here are the headers of the page:"
	            print response.info()
	    print "These are the cookies we have received so far:"
	    for i, cookie in enumerate(cj):
	        print i, ':', cookie
	    cj.save(COOKIEFILE)
	    return response.read()


在上一篇中我们已经实现了获取新闻版块、新闻标题和新闻URL这三类信息的函数，这一篇中我们将添加获取新闻内容的函数:

	def get_news_content(htmlstring):
	    content = None
	    # remove 'n' or 'r', otherwise the xpath will intrepret it
	    str = clean_html(htmlstring)
	    parsed_html = html.fromstring(str.decode('utf-8'))
	    # get inner text under comment node npm:article-content
	    inner_text_list = parsed_html.xpath("//founder-content/descendant::*/child::text()")
	    if inner_text_list:
	        inner_text = 'n'.join(inner_text_list)
	    else:
	        inner_text = ""
	    # get outer text under founder-content node
	    outer_text_list = parsed_html.xpath("//founder-content/child::text()")
	    # combine the outer text and inner text list
	    # it only have following combination:
	    # - outer_text_list[0] + inner_text + outer_text_list[1]
	    # - outer_text_list[0] + inner_text
	    # - inner_text
	    if not outer_text_list:
	        content = inner_text
	    if len(outer_text_list) == 1:
	        content = outer_text_list[0] + 'n' + inner_text
	    elif len(outer_text_list) == 2:
	        content = outer_text_list[0] + 'n' + inner_text + 'n' + outer_text_list[1]
	    else:
	        content = inner_text
	    return content

这段代码中同样通过xpath来获取页面中的新闻信息，通过我们手动进行firebug的分析，有效的新闻信息包含在如下格式的代码中:

	<founder-content style="font-size: 14px; line-height: 25px;">
	    <!--<npm:article-content>-->
	    ...
	    <!--</npm:article-content>-->
	    </founder-content>


这里的xpath语法有点意思，首先npm:article-content这个元素节点是comment node，不是普通的html element node,所以你在找到founder-content节点后，用getchildren()方法并不能定位到npm:article-content节点。怎么办？

我们分析到可能出现这么几种情况:

- 在```<founder-content>```和```<!--<npm:article-content>-->```之间存在text node, 并且在```<!--</npm:article-content>-->```和```</founder>```间也存在text node</li>


	    <founder-content> 
	    <text node> 
	    <!--<npm:article-content>--> 
	    <text node> 
	    <!--</npm:article-content>--> 
	    <text node> 
	    </founder-content>

- 只在```<founder-content>```和```<!--<npm:article-content>-->```之间存在text node

	    <founder-content> 
	    <text node> 
	    <!--<npm:article-content>--> 
	    <text node> 
	    <!--</npm:article-content>--> 
	    </found-content>


- 只在```<!--</npm:article-content>-->```和```</founder-content>```间存在text node


	    <founder-content>
	    <!--<npm:article-content>--> 
	    <text node> 
	    <!--</npm:article-content>--> 
	    <text node> 
	    </founder-content> 


我们做出如下定义：

- 在```<founder-content>```和```<!--<npm:article-content>-->```节点间的text节点, 定义为outer_text_front 
- 在```<!--</npm:article-content>-->```和```</founder-content>```节点间的text节点, 定义为outer_text_end 
- 在```<!--<npm:article-content>-->```和```<!--</npm:article-content>-->```节点间的text节点，定义为inner_text

不论在哪种情况，inner_text中是存在的，如果我们解析出的outer_text_list的数目只有一个，那就说明这个outer_text只可能是outer_text_front, 如果我们解析出的outer_text_list的数目有两个，那么第一个就是outer_text_front, 第二个就是outer_text_end, 我们将outer_text与inner_text按特定顺序拼接起来就是完整的新闻内容。

outer_text_list我们可以通过获取```<founder-content>```的直接子节点的text内容获取，inner_text_list我们可以通过获取```<founder-content>```的最内层子节点的text内容获取，这里我们假设在```<!--<npm:article-content>-->```和```<!--</npm:article-content>-->```之间只有一层text node，没有更深层次的text node, inner_text_list总是获取最深层次的text node,如果只有一层的话，它直接获取的是```<!--<npm:article-content>-->```和```<!--</npm:article-content>-->```间的内容。

好了，加上前篇所列的两个函数，主程序可以写成如下:

	if __name__ == '__main__':
	    init_robot()
	    index_page = fetch_url(get_absolute_url('index', date.today()))
	    sections = get_news_sections(index_page)
	    for section_title in sections:
	        print '新闻版块: %s' % section_title.encode('utf-8')
	        news = sections[section_title]
	        for news_title in news:
	            print 't新闻标题: %s' % news_title.encode('utf-8')
	            news_url = get_absolute_url('news', date.today(), news[news_title])
	            if news_url:
	                htmlstring = fetch_url(news_url)
	                if htmlstring:
	                    print 'tt%s' % get_news_content(htmlstring).encode('utf-8')


这里注意每次我们抓取的HTML内容要decode成python内部处理的unicode格式，然后输出时又要encode成utf-8来显示。 这里之所以用utf-8是因为我的terminal和当前locale都配置成了utf-8的格式的，如果你的terminal是gbk的，请编码成相应的gbk的。 好了，到此为止，所有的代码都提供并且分析了，下面是输出的部分内容，大家有兴趣在自己机器上也可以跑一跑！
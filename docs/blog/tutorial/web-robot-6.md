---
date: '2017-04-13 10:21:06'
draft: false
lastmod: '2017-11-25 20:16:11'
categories:
- 博客
tags:
- tutorial
- webrobot
title: Web抓取分析机器人连载(六)
---

上一篇中我们介绍了怎样用lxml和xpath来获取蓝色、绿色和棕色表格，这一篇我们将进入实质性的分析过程。 我们所抽取的信息的内容分为这么三种:

- 新闻版块名称
- 某新闻版块下的新闻标题
- 某新闻版块下的新闻URL


我们用python的字典来表示它们之间的关系。 A版块下的新闻列表['新闻标题A'] = 新闻的URL, 新闻版块列表[‘新闻版块A'] = A版块下的新闻列表。二话不说，翠花上代码， 下面代码展示了怎样解析新闻索引页面来获取新闻版块和其版块下的新闻列表信息:

	def clean_html(text):
	    special_chars= ['r', 'n', ' ']
	    for i in special_chars:
	        text = text.strip().replace(i, '')
	    return text
	    
	def get_news_sections(htmlstring):
        parsed_html = html.fromstring(htmlstring.decode('utf-8'))
        # by check the html source code of changsha web page
        # it can be see each section are surrounded by a outside table
        # with attributes <table width="710" cellspacing="0" cellpadding="0" border="0" style="margin-top: 10px; border: 1px solid #9D9C77;">
        outside_tables = parsed_html.xpath("//table[@width='710']")
        # I want fetch the section title and all the links in this section
        sections={}
        for table in outside_tables:
            # by checking the html source codes, the first table contains the section title
            fonts = table.xpath(".//table[1]/tr/td/font")
            if fonts:
                section = fonts[0]
                links={}
                for (element, attribute, link, pos) in table.iterlinks():
                        # This html.iterlinks method finds any link in an
                        # action, archive, background, cite, classid, codebase, data, href,
                        #longdesc, profile, src, usemap, dynsrc, or lowsrc attribute.
                        if attribute == "href":
                                links[clean_html(element.getchildren()[0].text)]=link
                sections[section.text] = links
        return sections


这里与上一篇有几个地方不同：

1. 这里只抽取了蓝色最外面的大的table,叫做outside_table
2. 在这个outside_table里面，我们直接用xpath来获取了其子table下的第一行第一列中的font标签中的内容，这个内容就是新闻版块信息
3. 在获取新闻版块后，我们没有用xpath来抽取棕色的子table,而是用lxml库中自带的iterlinks()方法来遍历outside_table下所有链接的信息 

特别提醒要注意的问题都用红颜色的文字标注下来了：

1. 在第二行， 我们抓取到的html内容，需要进行utf-8到unicode的编码。 在前面的教程中，我们知道该新闻网站的内容都是utf-8编码的，为了在python代码中操作方便，我们需要将utf-8编码的内容转换成unicode编码。
2. 在第十三行， 用xpath解析新闻版块信息时，注意xpath语法字符串中最前面的为"."字符，表示从当前获取的HTML元素也就是outside_table开始，而不是从html的跟元素开始解析。
3. 在第二十二行， 用lxml库的iterlinks()方法获取的链接有很多种，只要含有如下html属性的标签中的内容都被认为是链接：action， archive， background， cite， classid, codebase, data, href, longdesc, profile, src, usemap, dynsrc, or lowsrc。下面的HTML代码中，background=后面的内容也会被认为是链接，所以这里我们要区分不同的链接。 这里例子里面用attribute == "href" 来判断获取的链接是否是<a href="">的链接。

	</table><table width="710" border="0" cellspacing="0" cellpadding="0" style="margin-top: 10px;border: 1px solid #9D9C77;">
    <tr>
        <td width="678" height="24" background="../../../image/image04.jpg">

4. 在第二十三行，获取到新闻标题后，我们需要对其进行一些合适的字符过滤操作，比如过滤掉回车符、换行符、文字之间不必要的空格等等。

未完待续！
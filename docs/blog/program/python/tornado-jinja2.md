---
date: '2017-06-08 12:06:47'
draft: false
lastmod: '2017-11-25 20:16:11'
categories:
- 博客
tags:
- program
- python
title: Use Jinja2 as Tornado's default template engine
---

There are two ways to use Jinja2 as Tornado's default template engine:

- Inherited from mixed TemplateRendering class, and rewrite tornados's render() function
- Set Jinja2 template loader as the default template loader while initialize Application

#### For implemention 1:

Please refer to: [Using Jinja2 as the Template Engine for Tornado Web Framework](https://bibhas.in/blog/using-jinja2-as-the-template-engine-for-tornado-web-framework/)

#### For implemention 2:

Please refer to: [在Tornado中使用jinja2模版引擎的简单方法](https://zhu327.github.io/2015/01/02/%E5%9C%A8tornado%E4%B8%AD%E4%BD%BF%E7%94%A8jinja2%E6%A8%A1%E7%89%88%E5%BC%95%E6%93%8E%E7%9A%84%E7%AE%80%E5%8D%95%E6%96%B9%E6%B3%95/)
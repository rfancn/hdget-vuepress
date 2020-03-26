---
date: '2017-04-13 10:21:06'
draft: false
lastmod: '2017-11-25 20:16:11'
categories:
- 博客
tags:
- program
- web
title: why jquery.serialize() method missing some elements under form?
---

When submit a form by using jquery.serilize() method, sometimes it missing some control elements under form element, what caused that?

===

Just remember that form elements needs to have a name and a value to be serialized, if elements does not have a name, then it will not get serialized by jquery.

#### Reference:
- [stackoverflow](http://stackoverflow.com/questions/16881912/jquery-serialize-on-ajax-loaded-form-not-working)
---
date: '2017-04-13 10:21:06'
draft: false
lastmod: '2017-11-25 20:16:11'
categories:
- 博客
tags:
- program
- web
title: A guide to css font-family declarations for web design in simplified Chinese
---

This document is just a summarize of below document:
[Chinese Standard Web Fonts: A Guide to CSS Font Family Declarations for Web Design in Simplified Chinese ](http://www.kendraschaefer.com/2012/06/chinese-standard-web-fonts-the-ultimate-guide-to-css-font-family-declarations-for-web-design-in-simplified-chinese/)

1. Use the Chinese characters, and also spell out the font name, e,g:
     
         "Microsoft Yahei","微软雅黑"

2. Declare English target fonts before Chinese target fonts, e,g:

         English target fonts firstly: Tahoma, Helvetica, Arial 
         Then: "Microsoft Yahei","微软雅黑"

3. Declare the Microsoft font and the Mac font, e,g:

        Microsoft fonts: Microsoft YaHei New
          Mac IOS fonts: STXihei

As a conclusion, the recommended safe web fonts for English and Chinese mixed site are:

    @font-family-sans-serif:  Helvetica, Arial, "Microsoft Yahei", "微软雅黑", STXihei, "华文细黑", sans-serif;
    @font-family-serif:       Georgia, "Times New Roman", Times, serif;
 

- [Common fonts to all versions of Windows & Mac equivalents](http://www.ampsoft.net/webdesign-l/WindowsMacFonts.html)
- [Safe web fonts](http://web.mit.edu/jmorzins/www/fonts.html)
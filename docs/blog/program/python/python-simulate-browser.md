---
date: '2017-04-13 10:21:06'
draft: false
lastmod: '2017-11-25 20:16:11'
categories:
- 博客
tags:
- program
- python
title: Python simulate browser behavior without X
---

We can use python urllib, or python request library to help us download html page, and manually parse it to get what we expected.
But sometimes it is difficult to handle frontend javascript jump or server side redirect,
different javascript result in the different browser's behaivor, 
How to follow javascript logic and navigte to what we want to get is a challenge.
In this case, you can get an expected result in a easier way by using some automational functional testing tools, such as Selenium or Splinter.

- [splinter](http://splinter.cobrateam.info/)
- [selenium](http://seleniumhq.org/)

Here splinter project seems to be a wrapper on Selenium, to make it works on Oracle linux, you need do following:

	# yum install xorg-x11-server-Xvfb
	# yum install tigervnc-server
	# pip install pvvirtualdisplay

Basic python code:

	from pyvirtualdisplay import Display
	from splinter import Browser

	if __name__ == '__main__':
    	display = Display(visible=0, size=(1920, 1080))
	    display.start()

    	browser = Browser('firefox')
	    browser.visit("www.baidu.com")

    	print browser.html
---
date: '2017-04-13 10:21:06'
draft: false
lastmod: '2017-11-25 20:16:11'
categories:
- 博客
tags:
- program
- python
title: how to add 3rd party python packages or modules
---

类似于sosreport之类的module可以从/usr/lib/<python ver>/site-packages/<package name>直接导入并加以使用 e.g: import sos.policyredhat, 但是自己创建的package直接放入/usr/lib/<python ver>/site-packages/ 并不能直接import，提示module不存在错误。要使第三方创建的模块被其他python程序直接不带路径的调用，需要写好setup.py，并调用distutils来发布。

例如：我创建了一个test包，里面有一个子包plugins, 目录结构如下:

	<Your python source files dir>
	|-setup.py
	|-test
	    |---__init__.py
	    |---module1.py
	    |---moudle2.py
	    |------plugins
	    |       |-- __init__.py
	    |       |-- plugin1.py
	    |       |-- plugin2.py

你需要写如下一个setup.py,并执行python setup.py install来build和install这些包/模块文件到默认的/usr/lib/</python><python ver>/site-packages/目录

	from distutils.core import setup
	
	setup(
	    name = 'mytest',
	    packages = ['test', 'test.plugins'],
	)

Reference: [setupscript.html](http://docs.python.org/distutils/setupscript.html)
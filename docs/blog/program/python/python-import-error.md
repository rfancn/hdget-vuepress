---
date: '2017-04-13 10:21:06'
draft: false
lastmod: '2017-11-25 20:16:11'
categories:
- 博客
tags:
- program
- python
title: 'ImportError: No module named ''b.c'' while importing ''a.b.c'''
---

It spent me several hours to find out the root cause why the importing operation failed while importing 'a.b.c' module.

I wrote a function in __init__.py in 'a' package, I plan to get current module path use:

    self.cur_dir = __path__.pop()

In later logic, if I tried to load some module under this 'a' package, it always failed with below error:

    ImportError: No module named b.c

But it is fine when I try to load it via python shell or by other ways. Finally, I found the problem comes from this point: ``` __path__.pop()```, it pops and remove ```__path__``` info from global namespace, and next ```__import__``` function still parse the real module which need to load as 'b.c', but now it cannot find the ```__path__``` information, so it don't know where to load 'b.c' module, which in turn report 'No module named ...' error. Sigh! Never do any modify/write operation to global stuff unless you are very sure about it.
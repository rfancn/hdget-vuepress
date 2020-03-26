---
date: '2017-11-17 13:34:08'
draft: false
lastmod: '2017-11-25 20:16:11'
categories:
- 博客
tags:
- program
- golang
title: makefile in golang
---

#### Why
go install already checks if the binary is up-to-date before installation. go test is all you need to kick-off tests that run at face melting speed. It's natural to ask, "Why do I need a Makefile?" It because make is useful because it's executable documentation. It describes how to build your project, what kind of tests can be run and the external tools your project depends on and so on.



#### Reference:
- [Makefiles for Golang](https://sahilm.com/makefiles-for-golang/)
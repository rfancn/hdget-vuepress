---
date: '2017-06-08 12:06:47'
draft: false
lastmod: '2017-11-25 20:16:11'
categories:
- 博客
tags:
- program
- python
- celery
title: Tornado Celery Integration
---

#### Background
Current tornado-celery project seems doesn't support celery4.x, no update for this project for long time

#### Research

pika AMQP library already have an adpator "TornadoConnection" which can be used with the Tornado IO Loop. Then it has possibility to integrate this into celery to support async connection. Before starting with this target, we need to get familar with how celery send task in client side.

Assume we have a celery client use AMQP broker, and redis as result backend.
---
date: '2017-04-13 10:21:07'
draft: false
lastmod: '2017-11-30 16:59:49'
tags:
- project
- pygigo
title: Pygigo Internals - how to integrate and distributed deploy celery worker
categories:
- 项目
---

Before we dive into pygigo, pls let me introduce the way how to integrate and distribute depoly Django and Celery. In Celery tutorial, it explained in detail how to integrate Celery into Django, and the most important one is django-celery is not need after Celery 3.1 released. But in pygigo, what I considered is hwo to make Celery worker running on different servers. Maybe I am not a good google searcher, I cannot find a tutorial or document demostrate how to deploy and run Celery worker on different servers. However, this is the basic purpose of pygigo, I already put a very simple version of pygigo project on Github, anyone interest on it could search pygigo in Github and check it there.

#### Producer server Configuration
- Make sure firewall enable tcp connection for message broker's port, if server side use Redis as the message broker, then the default port 6379 need to be opened
- Install celery python package >=3.1, other nessary python packages will be installed automatically, except those, redis python package need to be installed too if you choose redis as the message broker
- Celery Configuraion(celeryconfig.py)

        BROKER_URL='redis://guest@{message broker server's hostname or ip}:6379//
        CELERY_RESULT_BACKEND = 'redis://{message broker server's hostname or ip}:6379/0'


#### Message Broker Server Configuration
- Make sure firewall enable outside connection to local message broker's port, if server side use Redis as message broker, then the default port 6379 need to be opened
- Install message broker on server host
- Configure message broker to allow foreign connection to itself, e,g: for Redis it need change below in /etc/redis.conf
        binding 0.0.0.0

#### Consumer(worker) Server Confiuration
- Install cerlery python package >=3.1, other nessary python packages will be installed automatically, except those, redis python package need to be installed too if you choose redis as the message broker
- Celery Configuration

        BROKER_URL='redis://guest@{message broker server's hostname or ip}:6379//'
        CELERY_RESULT_BACKEND = 'redis://{message broker server's hostname or ip}/0'

#### Example Codes
- Consumer server side(workers running here)
    - celeryconfig.py

            BROKER_URL='redis://guest@www.test.com:6379//'
            CELERY_RESULT_BACKEND = 'redis://www.test.com/0'

    - tasks.py

			from celery import Celery
            app=Celery()
            app.config_from_object('celeryconfig')

            @app.task
            def add(x, y):
                return x+y

- Producer server side
	- celeryconfig.py

			BROKER_URL='redis://guest@www.test.com:6379//'
			CELERY_RESULT_BACKEND = 'redis://www.test.com:6379/0'

#### Test Result
- start worker in consumer server as below

        # celery worker -A tasks -l info
		
- go to producer server side, and enter into python interactive mode, and execute below commands:

		$python
		>>> from celery import Celery
		>>> app=Celery()
		>>> app.config_from_object('celeryconfig')
		>>> result = app.send_task('tasks.add', (1,2))
		>>> result.state
		'SUCCESS'
		>>> result.get()
		3
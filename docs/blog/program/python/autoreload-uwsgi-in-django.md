---
date: '2017-04-13 10:21:06'
draft: false
lastmod: '2017-11-25 20:16:11'
categories:
- 博客
tags:
- program
- python
title: How to setup auto-reload uwsgi application in django dev environment
---

If we want to create django project with uwsgi, it is convenient to let uwsgi workers auto reload if there are some changes happens. Basically, we just need monitor the changes of *.py files in the project dir. For apache+mod_wsgi development environment, there is a [monitor.py](https://code.google.com/archive/p/modwsgi/wikis/ReloadingSourceCode.wiki#Restarting_Daemon_Processes) which called in wsgi.py can achieve the same target, but it looks like this method doesn't works well in nginx+uwsgi development, e,g: graceful reload of uwsgi workers could be affected when WSGI application executes monitor.py thread. 

Below method use inotify mechanism to monitor the project file(.py) changes under project dir, and update project dir to tell uwsgi service it need to restart workers.

        $ sudo pip install pyinotify

Add option 'touch-reload' in uwsgi specific config file to monitor which file/dir changes can cause uwsgi restart workers, e,g:

        project_home = /path/to/project
        touch-reload= %(project_home)

Download [uwsgi-monitory.py](https://github.com/rfancn/web-config/blob/master/uwsgi/uwsgi-monitor.py) script in django project directory, and run it with below syntax:

        $ python uwsgi-monitor.py [/path/to/project]

Or if uwsgi specific config file could be found in this project dir:

        $ python uwsgi-monitor.py
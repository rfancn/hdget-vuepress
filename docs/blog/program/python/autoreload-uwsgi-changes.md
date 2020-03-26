---
date: '2017-04-13 10:21:06'
draft: false
lastmod: '2017-11-25 20:16:11'
categories:
- 博客
tags:
- program
- python
title: Autoreload uWSGI when any changes made in Django project
---

There is a monitor.py can be used in Apache + mod_wsgi deployment, which can be used to auto reload wsgi processes when there is any change under Django project directory. I saw there are many way to achieve the same target in nginx + uWSGI deployment, like uwsgi.reload(), set "--py-autoreload=N" option. Finally I choose to use inotify to monitory the .py file changes under project dir, and touch the project dir to notify uWSIG reload python modules in workers.

- Add touch-reload option in project ini file

		touch-reload=/home/&lt;username&gt;/%(project)

- install pyinotify package

		(test) [ryan@localhost proj]$ pip install pyinotify

- create a mon_uwsgi.py file as following:

		#!/usr/bin/env python
		import signal
		import pyinotify
		import sys
		import os
		
		PID_FILE = '/tmp/mon_uwsgi.pid'
		LOG_FILE = '/tmp/mon_uwsgi.log'
		
		def handleExit(signum, stack):
		    try:
		        os.remove(PID_FILE)
		        os.remove(LOG_FILE)
		    except:
		        sys.exit(1)
		
		    sys.exit(0)
		
		class TouchEventHandler(pyinotify.ProcessEvent):
		    def my_init(self, target):
		        self.target = target
		
		    def __pflush(self, str):
		        sys.stdout.write(str+"n")
		        sys.stdout.flush()
		
		    def __touch_target(self):
		        try:
		            if os.path.exists(self.target):
		                os.utime(self.target, None)
		                self.__pflush("Successfully touch existed target:%s!" % self.target)
		            else:
		                open(self.target, 'a').close()
		                self.__pflush("Successfully touch new target:%s!" % self.target)
		        except Exception,e:
		            self.__pflush("Failed to touch target because of:%s!" % self.target)
		            return False
		
		        return True
		
		def process_default(self, event):
		    if not event.name.endswith(".py"):
		        return
		
		    self.__touch_target()
		
		def monitor(target):
		    if not os.path.exists(target):
		        print "Monitor Path: %s doens't exist!" % target
		        sys.exit(1)
		
		    signal.signal(signal.SIGINT, handleExit)
		    signal.signal(signal.SIGQUIT, handleExit)
		    signal.signal(signal.SIGTERM, handleExit)
		
		    event_handler = TouchEventHandler(target=target)
		    wm = pyinotify.WatchManager()
		    notifier = pyinotify.Notifier(wm, default_proc_fun=event_handler)
		    flags = pyinotify.IN_CREATE | pyinotify.IN_DELETE | pyinotify.IN_MODIFY | pyinotify.IN_MOVED_TO
		    wm.add_watch(target, flags, rec=True, auto_add=True)
		
		    try:
		        notifier.loop(daemonize=True, pid_file=PID_FILE, stdout=LOG_FILE)
		    except pyinotify.NotifierError, err:
		        raise err
		
		if __name__ == "__main__":
		    if len(sys.argv) &lt; 2:
		        print "python mon_wsgi.py &lt;project path&gt;"
		        sys.exit(1)
		
		    target_path = os.path.abspath(sys.argv[1])
		    print "Monitor for path: %s started, all changes under the dir will cause uWSGI reload!" % target_path
		    monitor(target_path)


Reference: [The Art Of Graceful Reloading of uWSGI](http://uwsgi-docs.readthedocs.io/en/latest/articles/TheArtOfGracefulReloading.html)
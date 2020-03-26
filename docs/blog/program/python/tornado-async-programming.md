---
date: '2017-04-15 21:03:22'
draft: false
lastmod: '2017-11-25 20:16:11'
categories:
- 博客
tags:
- program
- python
title: Tornado Asynchronous Programming Gists
---

- [Asynchronous programming with Tornado](https://gist.github.com/lbolla/3826189)

	-  Tornado is single threaded (in its common usage, although in supports multiple threads in advanced configurations), therefore any "blocking" task will block the whole server. This means that a blocking task will not allow the framework to pick the next task waiting to be processed. The selection of tasks is done by the IOLoop, which, as everything else, runs in the only available thread.

			# Example of misuse of callback. DON'T DO THIS!
			import time
			from tornado.ioloop import IOLoop
			
			
			def blocking_func():
			    print 'sleeping'
			    time.sleep(1)
			    print 'awake!'
			
			
			if __name__ == "__main__":
			    # Note that code is executed sequantially!
			    IOLoop.current().add_callback(blocking_func)
			    IOLoop.current().add_callback(blocking_func)
			    IOLoop.current().start()
			    
    - In Tornado, every function that has a "callback" argument can be used with gen.engine.Task. Beware though: being able to use Task does not make the execution asynchronous! There is no magic going on: the function is simply scheduled to execution, executed and whatever is passed to callback will become the return value of Task. 

			import time
			from tornado.ioloop import IOLoop
			from tornado import gen
			
			def my_function(callback):
			    print 'do some work'
			    # Note: this line will block!
			    time.sleep(1)
			    callback(123)
			
			
			@gen.engine
			def f():
			    print 'start'
			    # Call my_function and return here as soon as "callback" is called.
			    # "result" is whatever argument was passed to "callback" in "my_function".
			    result = yield gen.Task(my_function)
			    print 'result is', result
			    IOLoop.instance().stop()
			
			
			if __name__ == "__main__":
			    f()
			    IOLoop.current().start()

    - The new style coroutine decorator, inside of a coroutine where you want to call yield something(), something must either also be a coroutine or return a Future.
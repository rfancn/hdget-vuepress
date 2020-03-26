---
date: '2017-04-20 18:00:40'
draft: false
lastmod: '2017-11-25 20:16:11'
categories:
- 博客
tags:
- program
- python
- celery
title: tornado-celery-integration
---

##### Scenario
While implementing wxgigo project, I need create an agent which will receive the POST requests from Wechat server,  then relay to AMQP broker. In the appserver side, celery workers fetch messages from AMQP server and save result in result backend, finally agent will retrieve the result for result backend and return to Wechat server.

Agent:
	
		                             +----------------+                                                           
	                                 | Wechat Server  |                                                           
	                                 +----------------+                                                           
	                                          |                                                                   
	                                          |http post                                                          
	+-----------------------------------------|---------------------------------+                                 
	|  Wxgigo Agent Server          +-------------------+                       |                                 
	|                               |      nginx        |                       |                                 
	|                               +-------------------+                       |                                 
	|                                         |forward http post                |                                 
	|                                         |                                 |                                 
	|                 +--------------+ +--------------+  +--------------+       |                                 
	|                 | wxgigo agent | | wxgigo agent |  | wxgigo agent |       |                                 
	|                 +--------------+ +--------------+  +--------------+       |                                 
	+------------------------------------------|--------------------------------+                                 
	                                           |AMQP message                                                      
	                                           |                                                                  
	                                  +-----------------+                                                         
	                                  |   AMQP Broker   |                                                         
	                                  +-----------------+                                                         

#### Consideration
By comparing multiple python web framework benchmark, I prefer to choose Tornado as the base framework for it's async supportability, but to utilize this async feature for performance consideration, it need decouple all the steps to be async ready including connecting to broker, send message, receive message reply, retrieve result from backend...

- The initial version:

		import tornado
		from celery import Celery
		
		CELERY_APP = Celery()
		CELERY_APP.config_from_object('celeryconfig')
		
		def celery_call(task_name, *args):
		    try:
		        async_result = CELERY_APP.send_task(task_name, *args, retry=False)
		        response = async_result.get(CELERY_RESULT_TIMEOUT)
		    except TimeoutError:
		        print "Timeout Error" 
		    except Exception,e:
		        print e 
		
		    if isinstance(response, unicode):
		        response = response.encode('utf8')
		
		    return response
		    
		class MainHandler(tornado.web.RequestHandler):
	    	def post(self):
	        	response = celery_call('api.core.main', ({}, {}, self.request.body,))
	        	self.write(response)
	        	
		if __name__ == "__main__":
		    agent = tornado.web.Application([(r'/', MainHandler),])
		    agent.listen(8080)
		    tornado.ioloop.IOLoop.instance().start()

This version performance is very bad, on VPS with 1core+1G memory(already running redis+nginx+php-fpm...), it only serve 6~7 requests per second. There are four points which can block the process, idealy, we need make them all asynchronically.

1. connect to broker
2. send message to broker
3. wait for message reply
4. retrieve result from backend

To adaptor the sync operation of point1,2,3 to async one, it depends on AMQP client library Celery is using(kombu). But at the time writting this article, the [kombu](https://github.com/celery/kombu) doesn't support asynchronical style. There is a tornado-celery project which use [pika](https://github.com/pika/pika) AMQP client library to change this behavior, but it looks like not supporting celery 4.x at this moment. In fact, the most time consumping step is waiting and retrieve result from backend, if we can tune it to be asynchronically, then we could expect a big performance increasing.

- tasks.py

        from celery import Celery
        import time

        app = Celery()
        app.config_from_object('celeryconfig')

        @app.task
        def test(body):
            print body
            time.sleep(3)
            return body[2:5]
  

- version1(using ThreadPoolExecutor)

        from tornado import ioloop, web, gen
        from concurrent.futures import ThreadPoolExecutor
        from tornado.concurrent import run_on_executor

        import tasks


        class MainHandler(web.RequestHandler):
            executor = ThreadPoolExecutor(100)
            io_loop = ioloop.IOLoop.current()

            @run_on_executor
            def celery_call(self, task_name, *args, **kwargs):
                async_result = CELERY_APP.send_task(task_name, args=args, retry=False)
                return async_result.get()

            @gen.coroutine
            def post(self):
                result = yield self.celery_call("api.core.main", {}, {}, self.request.body)
                self.write("%s" % result)

        if __name__ == "__main__":
            agent = web.Application([(r'/', MainHandler),])
            agent.listen(8080)
            ioloop.IOLoop.instance().start()

Above version doesn't works well under the wrk highload(threads:4, concurrent:1000, duration: 30s), when it running for sometime, drain_events loop will continue for ever, I don't know how, but it would be something wrong in implementing ThreadPoolExecutor, so this is not a accept solution.


- version2(using tornoado.ioloop)
		
		from tornado import ioloop, web, gen
        from tornado.concurrent import TracebackFuture        
        import tasks
				
		def _on_result(future, async_result):
		    if async_result.ready():
		        future.set_result(async_result.result)
		    # if result is not ready, add callback function to next loop
		    else:
		        IOLoop.instance().add_callback(_on_result, future, async_result)
		
		def async_celerycall():
		    future = TracebackFuture()
		    callback = kwargs.pop("callback", None)
		    if callback:
		        IOLoop.instance().add_future(future,
		                                     lambda future: callback(future.result()))
		    async_result = tasks.test.delay('abcdefghi')
		    IOLoop.instance().add_callback(_on_result, future, async_result)
		    return future
		
		class MainHandler(web.RequestHandler):
		    @gen.coroutine
		    def post(self):
		        result = yield async_celerycall()
		        self.write("%s" % result)
	
		if __name__ == "__main__":
		    agent = web.Application([(r'/', MainHandler),])
		    agent.listen(8080)
		    ioloop.IOLoop.instance().start()
		    
With above version1 of codes, in the same env, it can serve 160~170 requests per second, ~30X throughput increased, what a incrediable improvement;) In fact, if we don't fetch result from backend, the throughput is 240~250 in this env, it looks 50% time spent on retrieve data from backend, it still has big room to improve.

- version3(using gevent)

        from tornado import ioloop, web
        from gevent import monkey
        monkey.patch_all()
        import tasks

        class MainHandler(web.RequestHandler):
            def on_result_ready(self, async_result):
                self.write(async_result.result)
                self.finish()

            @web.asynchronous
            def post(self):
                tasks.test.delay("abcdefghi").then(self.on_result_ready)

        if __name__ == "__main__":
            agent = web.Application([(r'/', MainHandler),])
            agent.listen(8080)
            ioloop.IOLoop.instance().start()

This version use the feature in Celery4.x which supports eventlet and gevent, if we setup on_result_ready callback while calling celery task, and it detects that our env are using eventlet or gevent, it will set GreenletDrainer to AsyncResult consumer automatically, and the drainer will automatically started and continue using connection.drain_events(timeout=1) to check if there is a message reply from broker or not. Here, it need make sure web.asynchronous decorator is set for the post() or get(). For self.finish() instrument will indicate the http connection is ready at that time, otherwise, it returns immendially before on_result_ready callback invoked. This kind of version seems has the similar performance as version2, it still need future testing to verify the result.
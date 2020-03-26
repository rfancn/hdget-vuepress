---
date: '2017-04-13 10:12:33'
draft: false
lastmod: '2017-11-25 20:16:11'
categories:
- 博客
tags:
- program
- python
- web
title: python-framework-for-c10k
---

##### C10k problem

How to configure operating systems and write code to support thousands of clients is what we called [C10k problem](http://www.kegel.com/c10k.html)

##### Candidates:

- [Django](https://www.djangoproject.com/)
    
    It is a high-level Python Web framework that encourages rapid development and clean, pragmatic design". If you are building something that is similar to a e-commerce site, then you should probably go with Django. It will get your work done quick. You dont have to worry about too many technology choices. It provides everything thing you need from template engine to ORM. It will be slightly opinionated about the way you structure your app, which is good If you ask me. And it has the strongest community of all the other libraries, which means easy help is available.

- [Flask](http://flask.pocoo.org/)
  
    It is a microframework for Python based on Werkzeug, Jinja 2 and good intentions". Beware - "microframework" may be misleading. This does not mean that Flask is a half-baked library. This mean the core of flask is very, very simple. Unlike Django, It will not make any Technology decisions for you. You are free to choose any template engine or ORM that pleases you. Even though it comes with Jinja template engine by default, you are always free to choose our own. As far as I know Flask comes in handy for writing APIs endpoints (RESTful services).

- [Twisted](https://twistedmatrix.com/)
  
    It is an event-driven networking engine written in python". This is a high-performance engine. The main reason for its speed is something called as deferred. Twisted is built on top of deferreds. For those of you who dont know about defereds, it is the mechanism through with asynchronous architecture is achieved. Twisted is very fast. But is not suitable for writing conventional webapps. If you want to do something low-level networking stuff, twisted is your friend.

- [Tornado](http://www.tornadoweb.org/en/stable/)
  
    It is a Python web framework and asynchronous networking library, originally developed at FriendFeed. By using non-blocking network I/O, Tornado can scale to tens of thousands of open connections, making it ideal for long polling, WebSockets, and other applications that require a long-lived connection to each user". Tornado stands some where between Django and Flask. If you want to write something with Django or Flask, but if you need a better performance, you can opt for Tornado. it can handle C10k problem very well if it is architected right.

- [Cyclone](http://cyclone.io/)
  
    It is a web server framework for Python that implements the Tornado API as a Twisted protocol". Now, what if you want something that is nearly as performant as Twisted but easy to write conventional webapps? Say hello to cyclone. I would prefer Cyclone over Tornado. It has an API that is very similar to Tornado. As a matter of fact, this is a fork of Tornado. But the problem is it has relativly small community. Alexandre Fiori is the only main commiter to the repo.

- [Falcon](https://falconframework.org/)
  
    Falcon is a very fast, very minimal Python web framework for building microservices, app backends, and higher-level frameworks. It is used by a growing number of organizations around the world, including Cronitor, EMC, Hurricane Electric, OpenStack, Opera Software, Wargaming, and Rackspace.The Falcon web framework encourages the REST architectural style, meaning (among other things) that you think in terms of resources and state transitions, which map to HTTP methods.

- [Sanic](https://github.com/channelcat/sanic)
  
    The concept for Sanic is simple: Provide a web framework with a syntax based on the widely used Flask, but build it on top of the wicked-fast asynchronous event handlers available in Python 3.5 and up. The result? A framework that can serve 33,000-plus requests per second to Flask's 4,988, and at almost one-tenth of the latency. (Benchmarks provided by the authors.) Routing and middleware are both included, too. It's not clear how readily Flask can be swapped out for Sanic in an existing app, but it's worth a try if you want to see how much of an improvement you can have.

- [Bottle](http://http://bottlepy.org)
  
    Bottle is a fast, simple and lightweight WSGI micro web-framework for Python. It is distributed as a single file module and has no dependencies other than the Python Standard Library.


##### Discussion

- [When to use Tornado, when to use Twisted / Cyclone / GEvent / other](http://stackoverflow.com/questions/13941903/when-to-use-tornado-when-to-use-twisted-cyclone-gevent-other)
- [Web Applications & Frameworks](http://python-guide-pt-br.readthedocs.io/en/latest/scenarios/web/)
- [Python's Web Framework Benchmarks](http://klen.github.io/py-frameworks-bench/)
- [TORNADO VS. GEVENT - BENCHMARKS](http://blog.kgriffs.com/2012/12/12/gevent-vs-tornado-benchmarks.html)
- [Benchmarking node, tornado and django for concurrency](https://swizec.com/blog/benchmarking-node-tornado-and-django-for-concurrency/swizec/1616)
- [Benchmark of Python WSGI Servers](http://nichol.as/benchmark-of-python-web-servers)

#### Consideration

My business logics is receiving HTTP Post requests from other servers with XML content, and then assmebly as AMQP message and send to AMQP Broker, then wait for reply or just returns quickly

- Tornado
- Flask
- bottle

Following framework are ruled out:

- Django: it is too heavy, though it is very good one with lots of documentation and community support, plents of plugins/extensions created against it
- Sanic: it is using Python3.5, I want to use python2.7
- Falcon: it is designed for REST architectural style
- Cyclone: it seems missing support from community,
- Twisted: it is a general purposed one, not only designed for web purpose

#### Test1(Echo 'hellow world')

I use [wrk](https://github.com/wg/wrk) to benchmark the http requests on Tencent cloud server.

- Tornado

	- codes:
	
			import tornado.ioloop
			import tornado.web
			
			class MainHandler(tornado.web.RequestHandler):
			    def get(self):
			        self.write("Hello World")
			
			if __name__ == "__main__":
			    agent = tornado.web.Application([(r'/', MainHandler),])
			    agent.listen(8080)
			    tornado.ioloop.IOLoop.instance().start()

	- test result
	
		    $ ./wrk -t10 -c1000 -d60s http://xxx:8080/
		    Running 1m test @ http://xxx:8080/
		      10 threads and 1000 connections
		      Thread Stats   Avg      Stdev     Max   +/- Stdev
		        Latency   253.88ms  384.25ms   1.98s    89.66%
		        Req/Sec    49.61     38.68   590.00     71.01%
		      27151 requests in 1.00m, 5.33MB read
		      Socket errors: connect 0, read 209, write 0, timeout 1350
		    Requests/sec:    452.04
		    Transfer/sec:     90.94KB

- gunicorn + Flask(prefork 4 workers)

	- codes

			from flask import Flask
			app = Flask(__name__)
			
			@app.route('/')
			def hello_world():
			    return 'Hello, World!'

	- test result

		    $ gunicorn -w 4 -b 0.0.0.0:8080 flask_hello:app
		    [2017-04-12 18:18:23 +0000] [23342] [INFO] Starting gunicorn 19.7.1
		    [2017-04-12 18:18:23 +0000] [23342] [INFO] Listening at: http://0.0.0.0:8080 (23342)
		    [2017-04-12 18:18:23 +0000] [23342] [INFO] Using worker: sync
		    [2017-04-12 18:18:23 +0000] [23347] [INFO] Booting worker with pid: 23347
		    [2017-04-12 18:18:23 +0000] [23348] [INFO] Booting worker with pid: 23348
		    [2017-04-12 18:18:23 +0000] [23355] [INFO] Booting worker with pid: 23355
		    [2017-04-12 18:18:23 +0000] [23358] [INFO] Booting worker with pid: 23358
		
		
		    $ ./wrk -t10 -c1000 -d60s http://119.29.193.127:8080/
		    Running 1m test @ http://119.29.193.127:8080/
		      10 threads and 1000 connections
		      Thread Stats   Avg      Stdev     Max   +/- Stdev
		        Latency   501.52ms  585.38ms   1.99s    81.79%
		        Req/Sec    36.89     51.90   380.00     85.70%
		      4284 requests in 1.00m, 723.76KB read
		      Socket errors: connect 0, read 2497, write 0, timeout 1576
		    Requests/sec:     71.30
		    Transfer/sec:     12.05KB
	
		!!! gunicorn worker got "WORKER TIMEOUT" and exitted abnormally

- gevent + Flask

	- code
		
			from flask import Flask,request
			from gevent.wsgi import WSGIServer
			from celerycall import celery_call
			
			app = Flask(__name__)
			
			@app.route('/', methods=['GET', 'POST'])
			def hello_world():
			    return 'Hello, World!'
			
			http_server = WSGIServer(('', 8080), app, log=None)
			http_server.serve_forever()

	- test result

		    $ ./wrk -t10 -c1000 -d60s http://119.29.193.127:8080/
		    Running 1m test @ http://119.29.193.127:8080/
		      10 threads and 1000 connections
		      Thread Stats   Avg      Stdev     Max   +/- Stdev
		        Latency   460.89ms  376.22ms   2.00s    86.06%
		        Req/Sec    47.70     28.34   560.00     77.03%
		      27537 requests in 1.00m, 3.39MB read
		      Socket errors: connect 0, read 257, write 0, timeout 2582
		    Requests/sec:    458.42
		    Transfer/sec:     57.84KB

- gevent + bottle

	- code
			
			from bottle import route, run, default_app
			from gevent.wsgi import WSGIServer
			
			@route('/')
			def hello():
			    return "Hello World!"
			
			app = default_app()
			http_server = WSGIServer(('', 8080), app, log=None)
			http_server.serve_forever()

	- test result

		    $ ./wrk -t10 -c1000 -d60s -s post.lua http://119.29.193.127:8080/
		    Running 1m test @ http://119.29.193.127:8080/
		      10 threads and 1000 connections
		      Thread Stats   Avg      Stdev     Max   +/- Stdev
		        Latency   407.74ms  358.20ms   1.99s    86.61%
		        Req/Sec    15.99     11.87   300.00     75.59%
		      7486 requests in 1.00m, 6.39MB read
		      Socket errors: connect 0, read 395, write 0, timeout 1370
		      Non-2xx or 3xx responses: 7486
		    Requests/sec:    124.63
		    Transfer/sec:    108.91KB

#### Test2: simulate post and headers sent from other server, it is a blocking request, Waiting for result

- blocked celery call

		from celery import Celery
		from celery.exceptions import TimeoutError
		
		CELERY_RESULT_TIMEOUT = 5
		
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
		    
- tornado 
	- code

			import tornado.ioloop
			import tornado.web
			from celerycall import celery_call
			
			class MainHandler(tornado.web.RequestHandler):
			    def post(self):
			        celery_call('api.core.main', ({}, {}, self.request.body,))
			
			if __name__ == "__main__":
			    agent = tornado.web.Application([(r'/', MainHandler),])
			    agent.listen(8080)
			    tornado.ioloop.IOLoop.instance().start()
		
	- test result

		    $ ./wrk -t4 -c1000 -d60s -s post.lua --latency http://xxx:8080/
			Running 1m test @ http://xxx:8080/
			  4 threads and 1000 connections
			  Thread Stats   Avg      Stdev     Max   +/- Stdev
			    Latency   567.96ms  514.98ms   1.64s    63.16%
			    Req/Sec     5.86     11.00    70.00     93.10%
			  Latency Distribution
			     50%  598.24ms
			     75%  623.31ms
			     90%    1.64s
			     99%    1.64s
			  416 requests in 1.00m, 58.50KB read
			  Socket errors: connect 0, read 1235, write 0, timeout 397
			Requests/sec:      6.92
			Transfer/sec:      0.97KB
	    
- gevent+Flask

	- code

			from flask import Flask,request
			from gevent.wsgi import WSGIServer
			from celerycall import celery_call
			
			app = Flask(__name__)
			
			@app.route('/', methods=['GET', 'POST'])
			def hello_world():
			    body = request.get_data()
			    celery_call('api.core.main', ({}, {}, body,))
			    return ""
			
			http_server = WSGIServer(('', 8080), app, log=None)
			http_server.serve_forever()
			
	- test result

		    $ ./wrk -t4 -c1000 -d60s -s post.lua --latency http://119.29.193.127:8080/
			Running 1m test @ http://119.29.193.127:8080/
			  4 threads and 1000 connections
			  Thread Stats   Avg      Stdev     Max   +/- Stdev
			    Latency   450.22ms  433.35ms   1.68s    90.00%
			    Req/Sec     6.18      9.86    70.00     92.02%
			  Latency Distribution
			     50%  628.19ms
			     75%  652.96ms
			     90%    1.17s
			     99%    1.68s
			  373 requests in 1.00m, 41.89KB read
			  Socket errors: connect 0, read 731, write 0, timeout 353
			Requests/sec:      6.21
			Transfer/sec:     713.77B


- gunicorn(3 workers) + Flask

	    $ ./wrk -t10 -c1000 -d60s -s post.lua http://xxx:8080/
	    Running 1m test @ http://xxx:8080/
	      10 threads and 1000 connections
	      Thread Stats   Avg      Stdev     Max   +/- Stdev
	        Latency     1.32s   592.30ms   1.78s    73.08%
	        Req/Sec     2.26      3.34    30.00     89.31%
	      446 requests in 1.00m, 69.25KB read
	      Socket errors: connect 0, read 1358, write 0, timeout 420
	    Requests/sec:      7.42
	    Transfer/sec:      1.15KB


#### Test3: simulate post and headers sent from other server, but not wait for result, so basically it works as a non-blocking style

- non-blocked celery call

		from celery import Celery
		from celery.exceptions import TimeoutError
		
		CELERY_RESULT_TIMEOUT = 5
		
		CELERY_APP = Celery()
		CELERY_APP.config_from_object('celeryconfig')
		
		def celery_call(task_name, *args):
		    try:
		        async_result = CELERY_APP.send_task(task_name, *args, retry=False)
		    except TimeoutError:
		        print "Timeout Error"
		    except Exception,e:
		        print e

- tornado

	- code: same as above, just the celery call changed	
    - test result
    
		    $ ./wrk -t4 -c1000 -d60s -s post.lua --latency http://xxx:8080/
			Running 1m test @ http://xxx:8080/
			  4 threads and 1000 connections
			  Thread Stats   Avg      Stdev     Max   +/- Stdev
			    Latency     1.35s   462.26ms   2.00s    64.72%
			    Req/Sec    78.33     65.91   640.00     71.26%
			  Latency Distribution
			     50%    1.43s
			     75%    1.76s
			     90%    1.88s
			     99%    2.00s
			  14627 requests in 1.00m, 2.01MB read
			  Socket errors: connect 0, read 605, write 0, timeout 11849
			Requests/sec:    243.50
			Transfer/sec:     34.24KB

- gevent+Flask

    - code: same as above, just the celery call changed	
	- test result
	
		    $ ./wrk -t4 -c1000 -d60s -s post.lua --latency http://119.29.193.127:8080/
			Running 1m test @ http://119.29.193.127:8080/
			  4 threads and 1000 connections
			  Thread Stats   Avg      Stdev     Max   +/- Stdev
			    Latency     1.30s   560.61ms   2.00s    63.56%
			    Req/Sec    71.75     49.59   393.00     68.24%
			  Latency Distribution
			     50%    1.44s
			     75%    1.80s
			     90%    1.90s
			     99%    1.99s
			  15186 requests in 1.00m, 1.67MB read
			  Socket errors: connect 0, read 581, write 0, timeout 10990
			Requests/sec:    252.82
			Transfer/sec:     28.39KB

#### Conclusion

From above tests, in a small server with limited memory, gevent really got some performance increasing when it choosen as WSGI server, gunicorn need adjust it's worker number to meet the VPS's capacity, otherwise it's performance is not so good. For blocking style, tornado seems is a bit better than gevent+Flask; but for non-blocking style, gevent+Flask gains < 5% performance increasing when compared with tornado. Anyway, either tornado and gevent+Flask is suitable for my project scenario.
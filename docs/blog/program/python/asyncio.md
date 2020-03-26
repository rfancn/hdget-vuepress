---
date: '2017-06-08 12:06:47'
draft: false
lastmod: '2017-11-25 20:16:11'
categories:
- 博客
tags:
- program
- python
title: python3.x asyncio study
---

#### How to convert sync codes to be async one

- Method1: 
	- create a future object
	- define a callback function which will continue check future result is ready or not, future.set_result() will be invoked when future result is ready, else schedule the callback to be called again in the even loop
	- in main function, start the callback function the first time, so it will continue executing until the future result is set
	- use await keyword to evaluate the future object until it is ready

	        event_loop = asyncio.get_event_loop()
	
			def retreive_from_db(future, id):
			    # it should have a async result or other flag which indicate we get the result from db or not
			    operation_flag = get_from_db(id)
			    # if the flag is ready, then we assume we already get real result, set the future to the real result
			    if operation_flag.ready():
			        future.set_result(real_result)
			    # else we scheduler to check operation done flag later
			    else:
			       event_loop.call_soon(retreive_from_db, future, id)
			       
			def main():
			    # create a future object and pass to callback function
			    db_result = asyncio.Future()
			    # scheduler the callback in event loop the first time
			    event_loop.call_soon(retreive_from_db, db_result, id)
			    # wait untile the db_result get the real result
			    final_result = await db_result

	Below is a rewrite of Tornado implementation of celery result retrieving in Sanic:
	
			from celery import Celery
			celeryapp = Celery()
			celeryapp.config_from_object('celeryconfig')
			
			app = Sanic(__name__)
			
			def retrieve_task_result(event_loop, task_result, async_result):
			    if async_result.ready():
			        task_result.set_result(async_result.get())
			    else:
			        event_loop.call_soon(retrieve_task_result, event_loop, task_result, async_result)
			
			async def send_task(request_body):
			    return celeryapp.send_task('tasks.test', (request_body,))
			
			@app.route('/', methods=['POST'])
			async def main(request):
			    request_body = request.body.decode('utf-8')
			    async_result = await send_task(request_body)
			
			    event_loop = asyncio.get_event_loop()
			    task_result = asyncio.Future()
			    event_loop.call_soon(retrieve_task_result, event_loop, task_result, async_result)
			    ret = await task_result
			    return text(ret)
			
			if __name__ == "__main__":
			    app.run(host="0.0.0.0", port=8080, debug=False)

- Method2
	- encapsulate the sync function to be Future object, in asyncio, it can use asyncio.ensure_future() or asyncio.get_event_loop().create_task(). Or you are expert, you can use asyncio.Task(somefunc(id)) directly
     	
	e,g: For below synchonize codes

			def handle(id):
			    subject = get_subject_from_db(id)
			    buyinfo = get_buyinfo(id)
			    change = process(subject, buyinfo)
			    notify_change(change)
			    flush_cache(id)
		   
	We can converted to below one:

			import asyncio
			import time
			
			async def get_subject_from_db(id):
			    time.sleep(3)
			    return "subject"
			
			async def handle(id):
				# turn blocked I/O function to be future object
			    subject = asyncio.ensure_future(get_subject_from_db(id))
			    buyinfo = asyncio.ensure_future(get_buyinfo(id))
			    # explicitly announce future task can be schedulered in event loop
			    results = await asyncio.gather(subject, buyinfo)
	     	    # explicitly announce process() can be schedulered in event loop
			    change = await process(results)
			    await notify_change(change) 
			    loop.call_soon(flush_cache, id)
		    
- Conclusion
	- encapsulate the sync function to be Future object, in asyncio, it can use asyncio.ensure_future() or asyncio.get_event_loop().create_task(). Or you are expert, you can use asyncio.Task(somefunc(id)) directly
	- use callback method, e,g:
		- asyncio.get_event_loop().call_soon(func, *args)
		- asyncio.get_event_loop().call_later(delay, func, *args)
		- asyncio.get_event_loop().call_at(when, func, *args)

References:
- [asyncio — Asynchronous I/O, event loop, and concurrency tools](https://pymotw.com/3/asyncio/)
- [使用Python进行并发编程-asyncio篇(一)](http://www.dongwm.com/archives/%E4%BD%BF%E7%94%A8Python%E8%BF%9B%E8%A1%8C%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B-asyncio%E7%AF%87/)
- [使用Python进行并发编程-asyncio篇(二)](http://www.dongwm.com/archives/%E4%BD%BF%E7%94%A8Python%E8%BF%9B%E8%A1%8C%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B-asyncio%E7%AF%87-%E4%BA%8C/)
- [使用Python进行并发编程-asyncio篇(三)](http://www.dongwm.com/archives/%E4%BD%BF%E7%94%A8Python%E8%BF%9B%E8%A1%8C%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B-asyncio%E7%AF%87-%E4%B8%89/)
---
date: '2017-04-15 21:03:21'
draft: false
lastmod: '2017-11-25 20:16:11'
categories:
- 博客
tags:
- program
- python
- celery
title: why celery AsyncResult on_result_ready callback not fired
---

There is a new feature in Celery4.x which introduce a Promise like style async method which can retrieve result asynchronicly. But I nerver make it works with RabbitMQ backend or Redis backend, so I dive into source codes to find out why it doesn't work.

When we call AsyncResult.then(on_result_ready), it will do following in celery_root/result.py

    def then(self, callback, on_error=None, weak=False):
        self.backend.add_pending_result(self, weak=weak)
        return self.on_ready.then(callback, on_error)
        
1. For ```self.backend.add_pending_result(self, weak=weak)```, it add myself(AsyncResult instance) into corresponding backend

    - for redis, it is backends/redis.py: RedisBackend
    - for rabbitmq, it is backends/rpc.py: RPCBackend
    
    There are two base BackendMixin has this ```add_pending_result```, SyncBackendMixin and AsyncBackendMixin, from celery4.0x, RedisBackend and RPCBackend all inherit from AsyncBackendMixin in async.py. Now we focus on what had been done in ```add_pending_result`` function.
    
        def add_pending_result(self, result, weak=False, start_drainer=True):
            if start_drainer:
                self.result_consumer.drainer.start()
            try:
                self._maybe_resolve_from_buffer(result)
            except Empty:
                self._add_pending_result(result.id, result, weak=weak)
            return result
            
	By default, it will automatically start the result consumer's drainer to try receive the messages from backend. At this moment, the self.result_consumer already initialited, the drainer comes from async.Drainer, the start() does nothing here. Then it will try get check wether this result already in pending results which still not return to consumer, if not found, then invoke ```async._add_pending_result()``` with result id and result.

	    def _add_pending_result(self, task_id, result, weak=False):
	        concrete, weak_ = self._pending_results
	        if task_id not in weak_ and result.id not in concrete:
	            (weak_ if weak else concrete)[task_id] = result
	            self.result_consumer.consume_from(task_id)

    Here _pending_result is a namedTuple (concrete, weak), here it does some sanity check and correction, we ignore to explain here, what's interesting is ```self.result_consumer.consume_from(task_id)```
    
    
        def consume_from(self, task_id):
        	if self._consumer is None:
                return self.start(task_id)

        	queue = self._create_binding(task_id)
        	if not self._consumer.consuming_from(queue):
            	self._consumer.add_queue(queue)
	            self._consumer.consume()
	            
	In fact, the self._consumer is still None, so it will create komobu Consumer and instruct it ready to consume messages from backend
	
	 - celery app connection channel
	 - queues = [task_id]
	 - callbacks = rpc.on_state_change
	 - others
	 
2. For ```self.on_ready.then(callback, on_error)```, it tells to call callback when AsycResult.on_ready


By reading above codes, the common pitfalls here is above steps create a consumer to get ready for consume results from backend, but in fact it is the ```conn.drain_events()``` that actually triggers the consuming. so even we called AsyncResult.then(on_result_ready), the callback ```on_result_ready``` will not be invoked until we trigger to drain_events from backend queue. AsyncResult.get() can trigger this callback with following logic:

1. result.py:           AsyncResult.backend.wait_for_pending
2. backend/async.py:    AsyncBackendMixin._wait_for_pending
3. backend/rpc.py:      RPCBackend.ResultConsumer._wait_for_pending
4. backend/async.py:    BaseResultConsumer._wait_for_pending
5. backend/async.py:    BaseResultConsumer.drain_events_until(result.on_ready, timeout=timeout, on_interval=on_interval)
6. backend/async.py:    Drainer.drain_events_until
7. backend/async.py:    AsyncBackendMixin.on_state_change: AsyncBackendMixin._get_pending_result(task_id) if state in states.READY_STATES
8. backend/async.py:    result.maybe_throw
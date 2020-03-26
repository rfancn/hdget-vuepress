---
title: tornado-coroutine
date: '2017-04-19 16:00:04'
draft: false
lastmod: '2017-11-25 20:16:11'
categories:
- 博客
tags:
- program
- python
- tornado
---

#### Preface
What's the difference between normal function and coroutine:

1. coroutines are functions that allow for multiple entry points, that can yield multiple times, and resume their execution when called again.
2. coroutines can transfer execution to any other coroutine instead of just the coroutine that called them.
3. Functions, being special cases of coroutines, have a single entry point, can only yield once, and can only transfer execution back to the caller coroutine.

#### Reference

The origin post comes from: [here](http://xidui.github.io/2016/01/26/%E6%B5%85%E6%9E%90tornado%E5%8D%8F%E7%A8%8B%E8%BF%90%E8%A1%8C%E5%8E%9F%E7%90%86/), it is a good post so I try to record it here with my understanding. At the same time, this [article](http://sahandsaba.com/understanding-asyncio-node-js-python-3-4.html) will help understand coroutine.

#### Content

There are many ways to run coroutine in tornado, this post will introduce how it works begin from IOLoop.run_sync function. The overview of workflow is below:

![workflow](tornado_loop.svg?classes=img-fluid)

The below code snippet is a good example to help understand demonstrate how coroutine works.  

	import random
	import time
	from tornado import gen
	from tornado.ioloop import IOLoop
	
	@gen.coroutine
	def get_url(url):
	    wait_time = random.randint(1, 4)
	    yield gen.sleep(wait_time)
	    print('URL {} took {}s to get!'.format(url, wait_time))
	    raise gen.Return((url, wait_time))
	    
	@gen.coroutine
	def outer_coroutine():
	    before = time.time()
	    coroutines = [get_url(url) for url in ['URL1', 'URL2', 'URL3']]
	    result = yield coroutines
	    after = time.time()
	    print(result)
	    print('total time: {} seconds'.format(after - before))
	    
	if __name__ == '__main__':
	    IOLoop.current().run_sync(outer_coroutine)

The outout for above example would be:

	URL URL1 took 1s to get!
	URL URL2 took 2s to get!
	URL URL3 took 2s to get!
	[('URL1', 1), ('URL2', 2), ('URL3', 2)]
	total time: 2.00353884697 seconds
	
#### coroutine
	
The common mistake is we assume while yield coroutine, the return value is a generator, in fact, each time we yield a object(it can be a normal function or another coroutine), it either returns a future object or coroutine:

1. The first time execute the real funtion, it should returns generator or final result
2. If it is still a geneartor(it means the final result still not get, the genearator need continue),
     - save previous stack context
     - execute the generator the second time by get ```generator.next()```
     - bypass 1st result, the initial future, 2nd result to Runner, which will scheduler again until we get final result
3. If it is a final value, set the future to final result, and return future. At this time, the iterator loop get stop.

		# gen.py
		def _make_coroutine_wrapper(func, replace_callback):
		    @functools.wraps(func)
		    def wrapper(*args, **kwargs):
		        future = TracebackFuture()
		        ...
		        try:
		            # 1st time result
		            result = func(*args, **kwargs)
		        ...
		        else:
		            if isinstance(result, types.GeneratorType):
		                try:
		                    orig_stack_contexts = stack_context._state.contexts
		                    # 2nd time result
		                    yielded = next(result)
		                    # if inside func, there exist yield keyword, then yielded(2nd time result) is a generator
		                    # if inside func, it invoke other coroutine, then yielded(2nd time result) is a future object
		                    ...
		                else:
		                    # Runner(1st time result, future, 2nd time result
		                    Runner(result, future, yielded)
		                try:
		                    return future
		                finally:
		                    future = None
		        future.set_result(result)
		        return future
		    return wrapper

#### future

Future is a specific object which used between scheduler and coroutine, it provides functions like: callback function registering(callback will be invoked after async events finish), tempoary result saving, child coroutine wake up parent coroutine(by Runner)... There is one future corresponding to one coroutine, each time we yield coroutine, it  returns a future object. What does future means: all the inner logic in coroutine finsihed, the result will be saved to future object, and invoke the callback specified in future object. Then the question is when the callback registered in Tornado, the answer is when you yield coroutine and return the future object.

	# ioloop.py IOLoop
	def run_sync(self, func, timeout=None):
	    future_cell = [None]
	    def run():
	        try:
	            result = func()
	        except Exception:
	            future_cell[0] = TracebackFuture()
	            future_cell[0].set_exc_info(sys.exc_info())
	        else:
	            if is_future(result):
	                future_cell[0] = result
	            else:
	                future_cell[0] = TracebackFuture()
	                future_cell[0].set_result(result)
	        self.add_future(future_cell[0], lambda future: self.stop())
	    self.add_callback(run)
	    if timeout is not None:
	        timeout_handle = self.add_timeout(self.time() + timeout, self.stop)
	    self.start()
	    if timeout is not None:
	        self.remove_timeout(timeout_handle)
	    if not future_cell[0].done():
	        raise TimeoutError('Operation timed out after %s seconds' % timeout)
	    return future_cell[0].result()
	    
	    
From above codes, it can be seen at very first time, it register a callback ```run```. Then in the next event loop, the callback ```run``` will be executed, and outer_coroutine ```func``` executed in turn, here the result is the future object of this outer_coroutine. If we don't do anything to this future object, then if this future object finishes, nothing will be happened, which will cause infinite loop. In fact,what we can see ```ioloop.add_future``` add a callback ```self.stop()``` to this future object, which make sure the coroutine finishes, the even loop will quit. By checking ```ioloop.add_future``` document, the purpose of it is schedules a callback on the IOLoop when the given Future is finished. **so the callback definied in future object will not be invoked immediatly when future finishes, there is one event loop gap between future object completetion and before executing the callback**

#### Runner

If we think tornada is a car, then Runner object is his engine, it is responsible for schedudling all coroutines to finish async events. One coroutine one Runner, each coroutine executed by one Runner. As we said before, coroutine can contain a inner coroutine(or may be a normal function, here we consider the complex one that outer coroutine contains inner coroutine), when yielding outer coroutine and returns a future object,  Runner will take over the inner future object while executing ```future=next(inner coroutine)```, and add it to ioloop. so the picture describes the order as: handle_yielded->run(There may be many io loops between them).

There are two important functions in Runner scheculer: handle_yielded and run. Let's check handle_yielded fistly:

	# gen.py Runner
	def handle_yield(self, yielded):
	    # Lists containing YieldPoints require stack contexts;
	    # other lists are handled via multi_future in convert_yielded.
	    if (isinstance(yielded, list) and
	            any(isinstance(f, YieldPoint) for f in yielded)):
	        yielded = Multi(yielded)
	    elif (isinstance(yielded, dict) and
	          any(isinstance(f, YieldPoint) for f in yielded.values())):
	        yielded = Multi(yielded)
	    if isinstance(yielded, YieldPoint):
	        ...
	    else:
	        try:
	            # conver yielded object to future object
	            self.future = convert_yielded(yielded)
	        except BadYieldError:
	            self.future = TracebackFuture()
	            self.future.set_exc_info(sys.exc_info())
	            
	    # if future is not done or it is in intermediate stage, make sure run() will be executed when future is finished       
	    if not self.future.done() or self.future is moment:
	        self.io_loop.add_future(
	            self.future, lambda f: self.run())
	        return False
	    return True
	    
In Runner, ```handle_yielded``` handles inner future object from inner coroutine. As most of events in coroutine are asynchonized, so most of the future object yields from inner coroutine are not finished, in this case, what can be done by Runner is register a callback when it receive this kind of future object.

Let's look at ```run``` function:

	# gen.py Runner
	def run(self):
	    """Starts or resumes the generator, running until it reaches a
	    yield point that is not ready.
	    """
	    if self.running or self.finished:
	        return
	    try:
	        self.running = True
	        while True:
	            future = self.future
	            if not future.done():
	                return
	            self.future = None
	            try:
	                orig_stack_contexts = stack_context._state.contexts
	                exc_info = None
	                try:
	                    value = future.result()
	                except Exception:
	                    self.had_exception = True
	                    exc_info = sys.exc_info()
	                if exc_info is not None:
	                    yielded = self.gen.throw(*exc_info)
	                    exc_info = None
	                else:
	                    yielded = self.gen.send(value)
	                if stack_context._state.contexts is not orig_stack_contexts:
	                    self.gen.throw(
	                        stack_context.StackContextInconsistentError(
	                            'stack_context inconsistency (probably caused '
	                            'by yield within a "with StackContext" block)'))
	            except (StopIteration, Return) as e:
	                self.finished = True
	                self.future = _null_future
	                if self.pending_callbacks and not self.had_exception:
	                    # If we ran cleanly without waiting on all callbacks
	                    # raise an error (really more of a warning).  If we
	                    # had an exception then some callbacks may have been
	                    # orphaned, so skip the check in that case.
	                    raise LeakedCallbackError(
	                        "finished without waiting for callbacks %r" %
	                        self.pending_callbacks)
	                self.result_future.set_result(getattr(e, 'value', None))
	                self.result_future = None
	                self._deactivate_stack_context()
	                return
	            except Exception:
	                self.finished = True
	                self.future = _null_future
	                self.result_future.set_exc_info(sys.exc_info())
	                self.result_future = None
	                self._deactivate_stack_context()
	                return
	            if not self.handle_yield(yielded):
	                return
	    finally:
	        self.running = False

It continous execute next() or send() operation against the generator passes to Runner(next() or send() all will iter geneartor to next stage, the only difference is send will pass a value to geneartor), if future result returns by generator is not finished and waiting for some response of async event, it will invoke ```handle_yielded()```. When the waited async event comes in, this ```run``` will be invoked because we already registered this ```run``` in previous ```handle_yielded```.

When generator quits, it means the coroutine in this Runner has been finished, it need set_result for the future object in this coroutine, also it indicates this future is ready and can call it's callback now. At this moment, the callback for the future is stop ioloop. For inner future object in inner coroutine, it corresponding to Runner's run in outer coroutine.

#### References

- [Combinatorial Generation Using Coroutines With Examples in Python](http://sahandsaba.com/combinatorial-generation-using-coroutines-in-python.html)
- [A Curious Course on Coroutines and Concurrency](http://dabeaz.com/coroutines/)
- [Improve Your Python: 'yield' and Generators Explained](https://jeffknupp.com/blog/2013/04/07/improve-your-python-yield-and-generators-explained/)
- [tornado.gen 模块解析](http://strawhatfy.github.io/2015/07/22/Tornado.gen/)
- [Tornado源码分析系列之一: 化异步为'同步'的Future和gen.coroutine](http://blog.nathon.wang/2015/06/24/tornado-source-insight-01-gen/)
- [tornado协程(coroutine)原理](http://blog.csdn.net/wyx819/article/details/45420017)
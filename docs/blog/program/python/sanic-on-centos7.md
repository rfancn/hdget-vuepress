---
date: '2017-06-08 12:06:47'
draft: false
lastmod: '2017-11-25 20:16:11'
categories:
- 博客
tags:
- program
- python
title: Install sanic on CentOS7
---

#### Why choosing sanic

All tests were run on an AWS medium instance running ubuntu, using 1 process. Each script delivered a small JSON response and was tested with wrk using 100 connections. Pypy was tested for Falcon and Flask but did not speed up requests. It can be seen sanic has the very good result, and basically it inspired by the idea behind document: [uvloop: Blazing fast Python networking](https://magic.io/blog/uvloop-blazing-fast-python-networking/)

| Server  |    Implementation   |Requests/sec|Avg Latency|
|---------|---------------------|------------|-----------|
| Sanic   | Python 3.5 + uvloop | 33,342     | 2.96ms    |
| Wheezy  | gunicorn + meinheld | 20,244     | 4.94ms    |
| Falcon  | gunicorn + meinheld | 18,972     | 5.27ms    |
| Bottle  | gunicorn + meinheld | 13,596     | 7.36ms    |
| Flask	  | gunicorn + meinheld | 4,988      | 20.08ms   |
| Kyoukai |	Python 3.5 + uvloop	| 3,889	     | 27.44ms   |
| Aiohttp |	Python 3.5 + uvloop	| 2,979	     | 33.42ms   |
| Tornado |	Python 3.5	        | 2,138	     | 46.66ms   |

#### Install latest python3 on CentOS7

	# yum install https://centos7.iuscommunity.org/ius-release.rpm
	# yum install python36u  python36u-pip
	
#### Install dependency tools/library which need to compile httptools, uvloop

	# yum install gcc python36u-devel
	
#### Install sanic

	# pip3.6 install sanic
	
#### Running sanic

	# python3.6 test.py
	
	test.py
	-------
	from sanic import Sanic
	from sanic.response import json
	
	app = Sanic()
	
	@app.route("/")
	async def test(request):
	    return json({"hello": "world"})
	
	if __name__ == "__main__":
	    app.run(host="0.0.0.0", port=80)
---
date: '2017-11-17 13:06:17'
draft: false
lastmod: '2017-11-25 20:16:11'
categories:
- 博客
tags:
- program
- web
title: CORS in Golang
---

#### Background

While I design wxgigo installer, I would like to get install configuration web via http, and then go through https to save install configuration into server, because the deploy information contains sensitive server access crendentials. Below is what I did:
```
...
function getHttpsUrl(pathname) {
    var withSlash = true;
    if (pathname.substring(0, 1) == "/") {
        withSlash = false;
    }
    var httpsUrl = "https://" + window.location.hostname;
    if (withSlash) {
        httpsUrl += "/" + pathname;
    }else{
        httpsUrl += pathname;
    }

    return httpsUrl
}
...
    $.ajax({
            url: getHttpsUrl("/install/save/"),
            dataType: 'json',
            contentType:"application/json; charset=utf-8",
            type: 'POST',
            data: JSON.stringify({
                'general': general.$data,
                ...
            }),
```

But if we just replace the http keyword with https keyword, and send ajax post request as above, it will get below failure:

```
Failed to load https://xxx.com/install/save/: Response to preflight request doesn't pass access control check: No 'Access-Control-Allow-Origin' header is present on the requested resource. Origin 'http://xxx.com:8080' is therefore not allowed access. The response had HTTP status code 404.
```

In fact, CORS problem was raised, how to enable CORS is what we will discuss here. CORS stands for Cross Origin Resource Sharing, and consists of a handshake of cryptic headers (usually) from a browser to a web api server. The handshake isn’t well known, and usually people just “enable” CORS in the web server, and all is right with the world again. Here is how the handshake breaks down:

![CORS flow](./cors_flow.png)

- Javascript Application initiates a XHR Request
- Browser intercepts request and sends the web server a “Preflight” request
	- Preflight consists of sending an OPTIONS request to the resource with “Origin” header set, and optionally other useful headers about the request which is to come, such as the Request Method and potential Request Headers
	- Server replies with the “Appropriate” response headers to the OPTIONS call stipulating what the browser should allow
- Browser gets the results of the Preflight and decides if the XHR Request should be allowed.
- Browser sends the request, or doesn’t



Reference: 
- https://husobee.github.io/golang/cors/2015/09/26/cors.html
- https://www.html5rocks.com/en/tutorials/cors/
---
title: "根据squid反向代理开启nginx服务器gzip"
date: "2012-07-05 17:04:51"
slug: "enable-nginx-gzip-by-squid-proxy-http-header"
published: true
author: "AlloVince"
legacy_id: 151
comment_status: "open"
comments: true
gitalk_id: "POST_151"
tags: []
---
状况大致是这样，服务商强制使用squid做网站的反向代理，用户过来的请求先进入squid反向代理，然后再转入我们的Nginx后端服务器。而引发的问题是设置开启的gzip失效了。

大概搜索了一下，发现引起这个问题的原因多半是由于反向代理对http1.0开启了gzip，而后端服务器的Nginx默认对http1.1开启gzip。

看一下请求，果然如此。

```
GET / HTTP/1.1
Host: shishijia.com
Accept-Encoding: gzip, deflate
Connection: keep-alive

HTTP/1.0 200 OK
Server: nginx/1.0.5
Date: Thu, 05 Jul 2012 08:28:39 GMT
Content-Type: text/html
Last-Modified: Thu, 05 Jul 2012 08:01:02 GMT
X-Cache: MISS from WT263CDN-3363
X-Cache-Lookup: MISS from WT263CDN-3363:80
Via: 1.0 WT263CDN-3363 (squid/3.0.STABLE20)
Connection: close
```

于是首先将Nginx的默认gzip http version更改为1.0

```
gzip  on;
gzip_http_version 1.0;
gzip_vary on;
```

gzip_vary的作用是在http响应中增加一行

```
Vary: Accept-Encoding
```

目的是改变反向代理服务器的缓存策略，反向代理服务器会根据后端服务器[是否带Vary头采用不同的缓存策略](http://rdc.taobao.com/blog/cs/?p=1490 "http://rdc.taobao.com/blog/cs/?p=1490")。

重启Nginx发现gzip依然没有开启。仔细看一下请求，那么可疑的部分还剩下X-Cache和Via。[http via](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html "http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html")是有代理服务器的特殊标头，会记录请求所经过的所有代理服务器信息，所以包含Via的请求就表示请求来自代理服务器而不是终端用户。

在Nginx的默认设置中，对于来自代理服务器是关闭的，可以通过

```
gzip_proxied         any;
```

打开。

刷新一下，[Nginx的gzip已经开启](/pages/enable-nginx-gzip-by-squid-proxy-http-header)了。

```
HTTP/1.0 200 OK
Content-Encoding: gzip
Connection: close
```

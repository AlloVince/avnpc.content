---
title: "对应Yslow的网站速度优化方法略谈"
date: "2008-08-12 16:21:11"
slug: "speed_up_website_with_yslow"
published: true
author: "AlloVince"
legacy_id: 110
comment_status: "closed"
comments: false
tags:
  - "Apache"
  - "CSS Sprites"
  - "mod_header"
  - "YD的程序员葛阁"
  - "Yslow"
---
**目录** [折叠]

1. [1\. Make fewer HTTP requests / 减少Http请求数](#toc_33688000)
2. [2\. Use a CDN / 使用CDN](#toc_33703800)
3. [3\. Add an Expires header / 为文件头指定Expires](#toc_33709700)
4. [4\. Gzip components / 启用Gzip压缩](#toc_33744800)
5. [5\. Put CSS at the top / 将Css文件放在头部](#toc_33758100)
6. [6\. Put JS at the bottom / 将Js文件放在底部](#toc_33762000)
7. [7\. Avoid CSS expressions / 避免CSS expressions](#toc_33765800)
8. [8\. Make JS and CSS external / 将Js和Css文件独立](#toc_33769700)
9. [9\. Reduce DNS lookups / 减少DNS查询](#toc_33773600)
10. [10\. Minify JS / 压缩Js文件](#toc_33777500)
11. [11\. Avoid redirects / 避免重定向](#toc_33787300)
12. [12\. Remove duplicate scripts / 移除重复的脚本](#toc_33804800)
13. [13\. Configure ETags / 配置ETags](#toc_33809000)

Yahoo!曾经针对网站速度体验提出了34条宝贵的准则《[Best Practices for Speeding Up Your Web Site](http://developer.yahoo.com/performance/rules.html)》，而[Yslow](https://addons.mozilla.org/zh-CN/firefox/addon/5369)正是按照这些准则，评测一个网站在速度体验上的优化程度的Firefox插件，将34条精简为更加直观的13条，并针对每一条给出从F\~A的评分以及最终的总分。

当然从评测得到的只能是一个分数以及建议，如何改进还是要靠自己，这里要谈的就是实实在在的如何针对每一条进行优化：

#### 1\. Make fewer HTTP requests / 减少Http请求数

一个网页不可避免的要引入大量的外部文件：Javascript、css、背景图片……由于Http协议的无状态性，用户的每一次访问，都会重新向服务器请求所有文件，而大量Http请求的累加，正是影响网站速度的最主要原因。

所以这里的解决方法只有一个：合并。最理想的情况莫过于一个网页只包含一个css，一个js，一张背景图。

合并Js和Css文件很好理解，背景图片要怎么合并？这里采用的主要方法是CSS Sprites，简单说就是把所有的图片拼接成一张大图，在不同的Css里指定背景图坐标来显示不同图片。具体可以参考Dave Shea的[Image Slicing’s Kiss of Death](http://alistapart.com/articles/sprites)一文，还有网站提供了[在线的CSS Sprites服务](http://spritegen.website-performance.org/)，只需要上传小图片，就可以获得拼接后的大图以及相应坐标。

不过在当前越来越多动辄包含10余个文件的开发框架面前，减少Http请求数也变得越来越难。一直都认为所谓框架，给出的应该是一整套完善的开发思想，从服务器配置到数据库设计甚至是到UI体验乃至SEO，但现在很多Framework总是各自为战，后台与前端脱节，只在自己的一片领域里提供一定程度上的方便，没有考虑到最终产品的统合，甚至连基本的代码侵入性问题没有处理好（这里点名批评dojo，恨不得在所有的html标签上印上dojo的章子），不能不说是一种遗憾。

所以如果网站中采用框架的话，在框架的选择面前，建议多采用轻量级，侵入性低的框架，也是为了日后产品的优化维护着想。

#### 2\. Use a CDN / 使用CDN

CDN(Content delivery network)内容分发网络，能够智能根据网络节点情况选择服务节点，大型网站部署时尤为重要。不过这属于硬件级别的解决方案，我们没有条件配置CDN的时候，可以自行设置忽略这一项评测。

在Firefox地址栏键入about:config，然后新建一个字符串，名称为extensions.firebug.yslow.cdnHostnames，值为所要评测网站的域名，多个设置用逗号分隔。例如我的设置就是allo.ave7.net,localhost

#### 3\. Add an Expires header / 为文件头指定Expires

Expires是浏览器Cache机制的一部分，浏览器的缓存取决于Header中的四个值： Cache-Control， Expires， Last-Modified， ETag。这个项目的考评主要针对Cache-Control和Expires。

具体的Cache原理不是本文所涉及的，有兴趣的同学可以看看[Caching Tutorial](http://www.mnot.net/cache_docs/)一文。为了优化这个选项，我们所要做的是对站内所有的文件有针对性的设置Cache-Control和Expires，这里基于Apache主机举例：

首先开启[mod_header](http://httpd.apache.org/docs/2.0/mod/mod_headers.html)模块，在httpd.conf中取消

```sql
LoadModule headers_module modules/mod_headers.so
```

一行的注释。然后对于图片，文件等不会经常更新的文件设置一个比较长的过期时间

```sql
<FilesMatch "\.(ico|pdf|flv|jpg|jpeg|png|gif|js|css|swf)$">
Header set Expires "Thu, 15 Apr 2010 20:00:00 GMT"
</FilesMatch>
```

对于Cache-Control可以设置的更加细致一些，这里我对图片，文件设置了1周，对XML，TXT设置了5小时，对html和php文件只设置了1小时。

```sql
<FilesMatch "\.(ico|pdf|flv|jpg|jpeg|png|gif|js|css|swf)$">
Header set Cache-Control "max-age=604800, public"
</FilesMatch>

<FilesMatch "\.(xml|txt)$">
Header set Cache-Control "max-age=18000, public, must-revalidate"
</FilesMatch>

<FilesMatch "\.(html|htm|php)$">
Header set Cache-Control "max-age=3600, must-revalidate"
</FilesMatch>
```

另外Expires也可以通过开启[mod_expires](http://httpd.apache.org/docs/2.0/mod/mod_expires.html)来实现，这里不再举例。

#### 4\. Gzip components / 启用Gzip压缩

HTTP/1.1支持接收服务器端经过[Gzip](http://www.gzip.org/)压缩的数据，在Apache2中，可以开启[mod_deflate](http://httpd.apache.org/docs/2.0/mod/mod_deflate.html)实现。

同样去掉注释

```sql
LoadModule deflate_module modules/mod_deflate.so
```

然后对所有文本类文件添加Gzip处理

```sql
DeflateCompressionLevel 3
<FilesMatch "\.(php|htm|html|js|css)$">
SetOutputFilter DEFLATE
</FilesMatch>
```

#### 5\. Put CSS at the top / 将Css文件放在头部

很好理解的一条，主要是为了避免最后加载Css引起的浏览器白屏，改善用户体验。

#### 6\. Put JS at the bottom / 将Js文件放在底部

同样很容易理解，为了让DOM先行加载。

#### 7\. Avoid CSS expressions / 避免CSS expressions

CSS expressions可以轻易的引起浏览器假死，也不在W3C规范内，不只是避免，最好完全不要用。

#### 8\. Make JS and CSS external / 将Js和Css文件独立

将Js和Css文件单独做成外部文件加载，一则可以功能复用，二则可以生成缓存，当然这一条和第一条要互相参照找出最好的解决方案才是。

#### 9\. Reduce DNS lookups / 减少DNS查询

外部文件分散于多个服务器，连接每台服务器都会做一次DNS查询，这一条是针对多服务器的部署。

#### 10\. Minify JS / 压缩Js文件

压缩Js文件，Yahoo!官方推荐的工具是[JSMin](http://crockford.com/javascript/jsmin)和[YUI Compressor](http://developer.yahoo.com/yui/compressor/)。

#### 11\. Avoid redirects / 避免重定向

每一次的重定向都会重新发送Header请求。所以在Apache下，无比强大的[mod_rewrite](http://httpd.apache.org/docs/2.0/mod/mod_rewrite.html)是必须要学的。

#### 12\. Remove duplicate scripts / 移除重复的脚本

开发中没有规划好，会出现页面中重复引用一个文件的情况，IE中即便是重复引用也会重新向服务器发送一次请求。

#### 13\. Configure ETags / 配置ETags

在第三条中已经对浏览器缓存机制中的Cache-Control和Expires进行了配置，这一条评测的是另外两个：Last-Modified和ETag

简单的说，即使设置了文件的期限，浏览器在访问资源时往往会因为Last-Modified和ETag而重新下载整个资源，所以简单的做法是关闭Last-Modified和ETag

在Apache中做如下配置

```sql
FileETag None

<FilesMatch "\.(ico|pdf|flv|jpg|jpeg|png|gif|js|css)$">
Header unset Last-Modified
</FilesMatch>
```

最后看看优化后的成果

![Yslow优化结果](https://lh4.googleusercontent.com/-HnywWsgUZaY/SKFHDtkIaZI/AAAAAAAAA68/Sb68B0JysCQ/Yslow.jpg?id=5233542371077548434)![Yslow优化结果2](https://lh6.googleusercontent.com/-dQJFzzUikio/SKTSqaQcP3I/AAAAAAAAA7I/2C43hqzvQBU/Yslow2.jpg?id=5234540292955979634)

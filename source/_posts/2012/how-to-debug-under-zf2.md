---
title: "Zend2(ZF2)的Debug及性能分析方法"
date: "2012-05-16 17:49:16"
slug: "how-to-debug-under-zf2"
published: true
author: "AlloVince"
legacy_id: 144
comment_status: "open"
comments: true
gitalk_id: "POST_144"
tags:
  - "Debug"
  - "Zend Framework 2"
  - "ZF2"
---
Zend1.0时代有非常棒的工具[ZFDebug](https://github.com/jokkedk/ZFDebug "https://github.com/jokkedk/ZFDebug"),但在ZF2下显然还没有什么太好的方法。

这里推荐老办法，用Xdebug + 分析工具，勉强可以分析ZF2执行效率和引用文件。

在php.ini内如下设置

```
zend_extension = "D:\xampp\php\ext\php_xdebug.dll"
xdebug.collect_includes = 1
xdebug.profiler_enable = 0
xdebug.profiler_enable_trigger = 1
xdebug.profiler_output_dir = "D:\xampp\tmp"
xdebug.profiler_output_name = "cachegrind.out.%u.log"
```

其中xdebug扩展的位置以及profiler的输出路径都需要根据实际情况调整。

配置完毕后重启Apache，在zf2项目URL中加入XDEBUG_PROFILE即可开启Xdebug Log输出，而平时则不会产生log。

```
http://localhost/?XDEBUG_PROFILE
```

输出log在windows下用[WinCacheGrind](http://sourceforge.net/projects/wincachegrind/ "http://sourceforge.net/projects/wincachegrind/")，在Linux下用[KCachegrind](http://kcachegrind.sourceforge.net/html/Home.html "http://kcachegrind.sourceforge.net/html/Home.html")打开即可。也可以用PHP实现的项目[Webgrind](https://github.com/jokkedk/webgrind "https://github.com/jokkedk/webgrind")。Webgrind的作者也是ZFDebug的作者。

---
published: true
date: '2012-05-01 00:02:40'
tags:
  - ZF2
  - Zend Framework 2
  - 资源
author: AlloVince
title: Zend Framework 2.0资料汇总
---

本日志整理并[汇总Zend Framework 2.0（下称ZF2）相关资料](http://avnpc.com/pages/zf2-summary)，不断更新中，欢迎补充：

Zend Framework 2.0 (ZF2)官方资源
======

- [Zend Framework 2（ZF2）官方网站](http://www.zendframework.com/)
- [Zend Framework 2（ZF2）用户手册](http://framework.zend.com/manual/2.0/en/index.html)
- [Zend Framework 2（ZF2）API文档](http://framework.zend.com/apidoc/2.0/namespaces/Zend.html)
- [Zend Framework 2（ZF2）模块汇总](http://modules.zendframework.com/)
- [Zend Framework 2（ZF2）Git代码库](https://github.com/zendframework/zf2/)
- [Zend Framework 2（ZF2）官方模块](https://github.com/zendframework)，ZF1的Service，Oauth等模块在ZF2的移植。
- [Zend Framework 2（ZF2）BUG汇报](https://github.com/zendframework/zf2/issues)

由于ZF2还在不断更新，比起从官方网站下载代码，更加推荐直接下载GIT库里的代码。


Zend Framework 2.0介绍及教程
======

首先可以通过下面的ppt对ZF2有一个全面的了解：

- [Introducing Zend Framework 2.0](http://www.slideshare.net/weierophinney/introducing-zend-framework-20)
- [Zend Framework 2.0 Patterns Tutorial](http://www.slideshare.net/weierophinney/zend-framework-20-patterns-tutorial)
- [Quick start on Zend Framework 2](http://www.slideshare.net/e.zimuel/quick-start-on-zend-framework-2)
- [Zend Framework 2.0 (zf2) 正式版发布及新功能介绍](http://avnpc.com/pages/zend-framework-2-0-released)

入门教程
--------

- 官方的[Getting Started with Zend Framework 2](http://packages.zendframework.com/docs/latest/manual/en/user-guide/overview.html)
- [ZF2入门：Windows环境下从零开始Zend Framework 2.0 (ZF2)环境搭建](http://avnpc.com/pages/zend-framework-2-installation-for-windows)
- [ZF2入门：Ubuntu/Linux环境下从零开始Zend Framework 2.0 (ZF2)环境搭建](http://avnpc.com/pages/zend-framework-2-installation-for-linux-ubuntu)


ZF2的一些新概念
--------

- 模块化：参考[ZF2 Modules Quickstart](http://mwop.net/blog/2012-09-19-zf2-module-screencast.html) ZF2模块快速入门
- DI：可以参考官方的[DI QuickStart](http://zf2.readthedocs.org/en/latest/modules/zend.di.quick-start.html)，但是我觉得这篇[来自Rob Allen的DI介绍](http://akrabat.com/zend-framework-2/an-introduction-to-zenddi/)更加容易理解一些，另外这里有位童鞋的举例很形象，[DI就是那万恶的包办婚姻](http://hi.baidu.com/irqiwxaanibiwzd/item/6d80b1ae41e717d95af1912e)啊：P
- EventManager：Yaodong Zhao童鞋翻译了官方手册的[Event Manager相关部分](http://dongbeta.com/2012/02/eventmanager-in-zend-framework-2/)
- ServiceManager：[Introduction to the Zend Framework 2 ServiceManager](http://blog.evan.pro/introduction-to-the-zend-framework-2-servicemanager)

如何学习
--------

ZF2由于本身的复杂性以及中文资源的稀缺，学习起来是有一定门槛。

所以建议有一定PHP基础的，对MVC有一定了解后再考虑使用ZF2框架。

ZF2大量使用了PHP的新特性，了解PHP5.3+有哪些新的特性和语法是必须的，比如[php匿名函数(Closure)](http://php.net/manual/zh/functions.anonymous.php)、[DateTime class](http://cn2.php.net/manual/zh/class.datetime.php)、[Locale class](http://cn2.php.net/manual/zh/class.locale.php)等都是经常用的。

对ZF2的几个重要的新概念，包括DI、EventManager、ServiceManager等，同样建议有所了解，可以对开发中采用正确的模式和方法起到辅助作用。

如果遇到问题，Google关键词寻求答案应该是开发者的本能。如果实在解决不了，可以考虑在以下几个地方提出问题：

- [Stack Overflow](http://stackoverflow.com/) 当然需要用英语提问，ZF2的作者也驻扎在上面，一个好问题也许可以得到非常好的解答。建议关注Stack Overflow的[zend-framework2](http://stackoverflow.com/questions/tagged/zend-framework2)标签看看一些问题是如何解决的。
- [德问](http://www.dewen.org/)和[SegmentFault](http://segmentfault.com/) 国内的编程问答社区，不过上面的Zend开发者都比较少。

笔者AlloVince在上述问答社区都有帐号，也欢迎邀请我作答。


Zend Framework 2 (ZF2)博客资源
=====

- [Matthew Weier O'Phinney](http://mwop.net/blog.html) ZF2主要作者
- [Evan Coury](http://blog.evan.pro/) 
- [Rob Allen](http://akrabat.com/) ZF2入门以及小技巧为主，官方的ZF2入门也是出自他手
- [Abdul Malik Ikhsan](http://samsonasik.wordpress.com/) 比较短的基础入门为主
- [Adam Lundrigan](http://adam.lundrigan.ca/)
- [Michael Gallego](http://www.michaelgallego.fr/blog/) 进阶实例比较多，需翻墙...
- [Jurian Sluiman](http://juriansluiman.nl/en/)
- [AlloVince 中文](http://avnpc.com/) 也就是笔者的Blog，原创发布一些ZF2的介绍和教程为主，支持[RSS订阅](http://avnpc.com/feed/)
- [Jacky.Wang的个人空间](http://my.oschina.net/ohcoding)
- [yangfan.co](http://yangfan.co) 


Zend Framework 2 (ZF2) 实例程序
====

- ZF2的Hello World实例可以直接参考官方的[ZendSkeletonApplication](https://github.com/zendframework/ZendSkeletonApplication)
- 简单的带数据库操作的实例参看[zf2-tutorial](https://github.com/akrabat/zf2-tutorial)
- 复杂的实例建议去[模块汇总](http://modules.zendframework.com/)中查找
- 笔者在运维的项目[EvaEngine](http://avnpc.com/pages/eva-engine)也是一个基于ZF2的大型应用


Zend Framework 2 (ZF2) 其他资源
====

- [比较全面的ZF2路由Router介绍PPT](http://stuff.dasprids.de/slides/zendcon2012/introducing-the-new-zend-framework-2-router/presentation.html)

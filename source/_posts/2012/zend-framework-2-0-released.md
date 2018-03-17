---
slug: zend-framework-2-0-released
date: '2012-09-06 18:23:55'
title: Zend Framework 2.0 (zf2) 正式版发布及新功能介绍
id: 158
tags:
  - ZF2
  - Zend Framework 2
---

著名php开源框架Zend Framework经历了长达数年的开发，终于在2012年9月5日正式发布了[2.0版本](http://avnpc.com/pages/zend-framework-2-0-released)，下简称ZF2。时隔Zend 1.0版本的发布已经有5年之久。 

php的框架一直都是百家争鸣的，但是作为php官方运维的框架，Zend Framework在php开发者中的流行度并不高。其一是由于Zend Framework主要面向大型应用，对代码规范以及程序结构有严格的规定，入门门槛较高。另一方面还是因为Zend Framework整体的执行效率偏低，略显笨重。

所以Zend小组本次可谓痛定思痛，ZF2并不像其他框架只是修修补补的更新，而是完全重写了ZF1的所有代码，主打的口号就是“高性能”。

来看一看ZF2都带开了哪些新的特性：


1. 模块化（ModuleManager）
-------------------------

比起ZF1来说，ZF2原生支持模块的概念，任意第三方php程序，只要遵循Zend的编码规范和代码结构，都可以变成一个Zend模块。目前已经有一批试验阶段的模块出炉（参看[ZF2 Modules](http://modules.zendframework.com/)）。 其中不乏像Doctrine ORM这样优秀的项目。

可以预见的是，随着ZF2的慢慢成熟，越来越多可以选择的模块将大量涌现，可能未来基于ZF2的项目开发，会像搭积木一样轻松简单。



2. 事件驱动（EventManager）
-----------

传统程序中，代码都是按线性顺序执行的，所以开发中往往很难将一些功能独立为一个组件或模块。

事件驱动，或者也可以叫钩子（Hook），改变了普通程序流程化的运行方式，应用了事件驱动之后，程序将呈现“注册事件” => “触发事件”的跳跃式运行，可以在不影响原有程序代码的，很容易的在任意位置加入新的业务逻辑，让项目的开发变得极为灵活。

3. 服务管理器（ServiceManager）
-----------

服务管理器的概念来自于"服务定位模式（[Service locator pattern](http://en.wikipedia.org/wiki/Service_locator_pattern)）"的编程思想。这种思想提倡将程序中的每一个独立功能提取出来作为一个“服务”，每一个服务都是独立可唤醒的，只有服务被调用时，服务相关的程序才会启动。

这也就是ZF2性能提升的秘密所在，[ZF2的MVC启动流程](http://avnpc.com/pages/zf2-mvc-process)中无处不体现ServiceLocator的思想，功能模块的调用极为“吝啬”，想必会给以前对Zend性能有意见的开发者一个大大的惊喜。

4. 依赖注入（Di Dependency Injection)
-----------

依赖注入广泛应用于Java的主流框架中，可以很好的解除大型应用中的耦合。ZF2引入Di也经过了反复的考量和权衡，即使进入beta阶段，Di仍然一度作为ZF2的基本实现方案，整个Mvc的配置基于Di。最终为了避免陷入Di可能造成的元数据式编程泥潭（[Metaprogramming](http://en.wikipedia.org/wiki/Metaprogramming)）,
Di只是作为ZF2的底层实现，上层加入了ServiceManager。普通开发者在使用ZF2的过程中不需要接触到Di的层面。不过这并不妨碍DI作为一个优秀的php组件存在并发挥作用。

5. 社会化编程
-----------

[ZF2的代码](https://github.com/zendframework/zf2/)完全托管在Github，借助Github的优秀设计，任何人都可以轻松的通过fork参与ZF2的项目建设，甚至提交新的模块功能。笔者[AlloVince](http://avnpc.com/)也帮助Zend小组修复了一些BUG，发现Zend小组响应非常快（从没有超过24小时），对反馈的意见也会花时间认真解答。所以参与ZF2项目是参与php开源项目一个不错的选择。


总结
-------

正如ZF2[发布信息](http://framework.zend.com/blog/zend-framework-2-0-0-stable-released.html)中写到的，没有哪个框架是完美的，ZF2也不例外。所以作为开发者要做的，不应该是纠结于哪个框架好哪个框架不好这种永远也得不到结论的问题，而是针对不同的项目选择合适的框架。

在现阶段，开发大中型php应用，特别是商业应用和企业应用，ZF2是一个非常不错的选择，因为ZF2有严格代码规范，非常适合团队开发。而ZF2作为Zend官方的支持产品，整体的可靠性和BUG的响应速度也都是有保证的。


相关资源
----------

最后对于有兴趣的朋友，欢迎访问[ZF2官方网站](http://framework.zend.com/)尝鲜。

笔者也整理出一些实用的[ZF2资源](http://avnpc.com/pages/zf2-summary)如下：

- [Zend Framework 2（ZF2）项目主页](http://framework.zend.com/)
- [Zend Framework 2（ZF2）用户手册](http://framework.zend.com/manual/2.0/en/index.html)
- [Zend Framework 2（ZF2）API文档](http://framework.zend.com/apidoc/2.0/namespaces/Zend.html)
- [Zend Framework 2（ZF2）模块汇总](http://modules.zendframework.com/)
- [笔者的Blog,会发布一些ZF2的技术日志](http://avnpc.com/)
- [Zend Framework 2.0的Mvc结构及启动流程分析（中文）](http://avnpc.com/pages/zf2-mvc-process)
- [Zend Framework 2 中的EventManager的使用方法(中文)](http://dongbeta.com/2012/02/eventmanager-in-zend-framework-2/)
- [EvaEngine:基于Zend Framework 2（ZF2）实现的大型应用，开发中](https://github.com/AlloVince/eva-engine)
- [Zend Framework 2.0数据库操作用例（中文）](http://avnpc.com/pages/advanced-database-select-usage-in-zf2)

更全面的资源请参看[Zend Framework 2.0资料汇总](http://avnpc.com/pages/zf2-summary)

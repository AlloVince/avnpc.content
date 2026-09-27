---
title: "Zend Framework 2.0的Mvc结构及启动流程分析"
date: "2012-05-22 13:46:48"
slug: "zf2-mvc-process"
published: true
author: "AlloVince"
legacy_id: 146
comment_status: "open"
comments: true
gitalk_id: "POST_146"
tags:
  - "MVC"
  - "Zend Framework 2"
  - "ZF2"
---
−目录

- [概要](#%E6%A6%82%E8%A6%81)

- [第一部分：初始化ServiceManager](#%E7%AC%AC%E4%B8%80%E9%83%A8%E5%88%86_%E5%88%9D%E5%A7%8B%E5%8C%96servicemanager)

- [第二部分：初始化模块](#%E7%AC%AC%E4%BA%8C%E9%83%A8%E5%88%86_%E5%88%9D%E5%A7%8B%E5%8C%96%E6%A8%A1%E5%9D%97)

- [第三部分：启动MVC](#%E7%AC%AC%E4%B8%89%E9%83%A8%E5%88%86_%E5%90%AF%E5%8A%A8mvc)

  - [Bootstrap引导阶段](#bootstrap%E5%BC%95%E5%AF%BC%E9%98%B6%E6%AE%B5)

    - [Zend\View的构成](#zend_view%E7%9A%84%E6%9E%84%E6%88%90)

    - [Zend\View的初始化](#zend_view%E7%9A%84%E5%88%9D%E5%A7%8B%E5%8C%96)

  - [MVC启动阶段](#mvc%E5%90%AF%E5%8A%A8%E9%98%B6%E6%AE%B5)

    - [Route路由启动](#route%E8%B7%AF%E7%94%B1%E5%90%AF%E5%8A%A8)

    - [Dispatch分发过程](#dispatch%E5%88%86%E5%8F%91%E8%BF%87%E7%A8%8B)

  - [发送最终响应并结束MVC](#%E5%8F%91%E9%80%81%E6%9C%80%E7%BB%88%E5%93%8D%E5%BA%94%E5%B9%B6%E7%BB%93%E6%9D%9Fmvc)

# 概要

首先需要明确的是，Zend2.0的启动以及MVC构架是完全基于事件驱动的。如果对事件驱动还不太了解的话，应该先弄清楚[什么是ZF2的事件驱动](../../pages/zf2-mvc-process "http://avnpc.com/pages/zf2-mvc-process")，并掌握基本的EventManager用法，这是阅读本文的基础。请参考[Using the ZF2 EventManager](http://mwop.net/blog/266-Using-the-ZF2-EventManager.html "http://mwop.net/blog/266-Using-the-ZF2-EventManager.html")。

基于事件驱动MVC与传统的MVC有什么不同，简单说就是由传统的复杂流程式调用过程。变成了先在某处注册事件，然后在某处触发事件的简单二元关系，事件不受代码结构和调用流程的影响，可以方便的解除耦合。

而在[最近才引入的ServiceManager](../../pages/zf2-new-changes-of-servicemanager "http://avnpc.com/pages/zf2-new-changes-of-servicemanager")也是Zend1中完全没有的概念，在我的理解来看，ServiceManager的引入是ZF2开发小组对于引入Di可能带来的元数据式编程问题（[Metaprogramming](http://en.wikipedia.org/wiki/Metaprogramming "http://en.wikipedia.org/wiki/Metaprogramming")）的一种[反思](http://zend-framework-community.634137.n4.nabble.com/Services-Instances-Dependencies-in-ZF2-td4584632.html "http://zend-framework-community.634137.n4.nabble.com/Services-Instances-Dependencies-in-ZF2-td4584632.html")。

ServiceManager带来的好处是：

- 将整个Zend构架的主要部分形象化，让结构更有组织，更利于理解

- 简化Di的配置，降低学习成本

- 进一步弱化了Bootstrap，让整个启动过程更加简洁

ServiceManager带来的不好之处是：

- 将Di做一层封装，无法直接通过配置文件控制整个构架

- 自定义需求比较高的时候，反而提高了学习成本，因为在学习Di的同时还要学习ServiceManager

那么闲聊至此，开始进入真正的[Zend2.0 MVC构架流程分析](../../pages/zf2-mvc-process "http://avnpc.com/pages/zf2-mvc-process")，这里以5月21日的[ZendSkeletonApplication](https://github.com/zendframework/ZendSkeletonApplication "https://github.com/zendframework/ZendSkeletonApplication")为例：

# 第一部分：初始化ServiceManager

ZendSkeletonApplication/public/index.php

```php
$configuration = include 'config/application.config.php';
$serviceManager = new ServiceManager(new ServiceManagerConfiguration($configuration['service_manager']));
$serviceManager->setService('ApplicationConfiguration', $configuration);
```

读取整个应用的基础配置文件，初始化Mvc框架所需要的ServiceManager。

这个过程中默认所依赖的所有类都写在Zend\Mvc\Service\ServiceManagerConfiguration中。ServiceManager的内部被划分为5类

1. services 服务

2. factories 工厂

3. abstractFactories 抽象工厂

4. aliases 别名

5. shared 共享服务

项目的配置文件application.config.php会复写Zend的默认配置并载入，比如如果需要使用一个自定义的服务，可以在配置文件中这样写

```php
<?php
return array(
    'service_manager' => array(
		'use_defaults' => true,
		'services' => array(
			'ViewManager'                  => 'EvaEngine\Mvc\View\ModuleViewManager',
		),
    ),
);
```

# 第二部分：初始化模块

ZendSkeletonApplication/public/index.php

```php
$serviceManager->get('ModuleManager')->loadModules();
```

ServiceManager中的ModuleManager，本质上是对Zend\Mvc\Service\ModuleManagerFactory的一个封装，主要做的工作包括：

1. 获得项目配置文件中需要载入的模块列表

2. 按配置遍历模块，分别载入模块的配置文件

3. 合并模块的配置文件

在配置文件中，可以通过modules节点控制具体载入哪些模块。

模块的载入同样采用了事件驱动，通过模块管理器Zend\ModuleManager\ModuleManager配合模块事件Zend\ModuleManager\ModuleEvent实现，在载入模块的过程中会依次触发

1. loadModules.pre 所有模块载入前

2. loadModule.resolve 每个模块载入

3. loadModule 每个模块载入后

4. loadModules.post 所有模块载入后

# 第三部分：启动MVC

终于到了MVC部分，整个MVC的流程都伴随着事件驱动，ZF2将其定义为MVC事件，按照执行顺序依次包括：

1. bootstrap 引导

2. route 路由

3. dispatch 分发

4. render 渲染

5. finish 结束

所以为了方便说明，将

ZendSkeletonApplication/public/index.php的

```php
$serviceManager->get('Application')->bootstrap()->run()->send();
```

拆分为三个阶段

## Bootstrap引导阶段

```php
$app = $serviceManager->get('Application')->bootstrap();
```

在Zend1中，Bootstrap曾经是MVC的核心部分，在ZF2中，由于事件驱动的引入，这一部分变得非常简单清晰：

首先在Zend\Mvc\Application→bootstrap()中，注册了所有MVC事件，初始化MvcEvent（将Request/Response/Router等注入），同时触发bootstrap事件。

这一过程中，View部分的初始化相对复杂，单独说明如下

### Zend\View的构成

在ZF2中，View部分同样做了非常大的改动，将Layout，Helper都合并入View。在Zend1中，Layout是一个独立存在的组件，而ZF2中将Layout和Template统一称为ViewModel，ViewModel是树形结构，这样就可以实现模板的递归嵌套，而在ZF2中的Layout，本质上就是位于树形结构最底层的ViewModel。

ZF2的View由以下几个部分组成，称呼是[AlloVince](../../ "http://avnpc.com/")个人的翻译，不当之处还请指正:

- View\View 视图，主要接管MVC事件

- View\Strategy 策略器，统筹安排视图的主要容器Placeholders，同时会将视图的最终结果放入容器，拼合为最后呈现给用户的内容

- View\Resolver 决策器，定义模板命名与实际路径的映射关系，同时决定模板最终对应的实际文件

- View\Renderer 渲染器，在决策器的辅助下，将ViewModel转换为文本输出。一个渲染器必须对应一个决策器才能工作。

- View\Model 视图模型，包括了视图中可能用到的所有变量。自身为树形结构，一个视图模型可以包含若干子模型

- View\Helper 视图助手，辅助生成HTML标签

在MVC构架中，Zend\Mvc\View\ViewManager会整合上述所有部分，最终构成整个视图。

### Zend\View的初始化

回到上一节，在bootstrap事件被触发时，视图部分做了一些主要的准备工作，包括：

- 指定一个MVC专用的策略器Zend\Mvc\View\DefaultRenderingStrategy，在这个策略器中将最顶层的ViewModel重定义Layout。注册MvcEvent::EVENT_RENDER事件

- 注入模板监听Zend\Mvc\View\InjectTemplateListener，最主要的作用是通过Controller和Action的名字来生成默认的视图名

- 注入视图模型监听Zend\Mvc\View\InjectViewModelListener

那么其实我们可以得出结论，Zend的Mvc中在bootstrap阶段，视图的所有准备工作都已经就绪了，并没有等到路由结束或者Controller启动。这样做的用意在于当路由失败时，仍然可以有对应的视图来呈现异常结果。

## MVC启动阶段

ZendSkeletonApplication/public/index.php

```php
$response = $app->run();
```

启动阶段对应的事件有

1. route 路由

2. dispatch 分发

如果异常发生，则会提前结束启动过程，分发事件有可能不会触发而直接触发finish(结束)事件。

### Route路由启动

ZF2的路由最有意义的重构是允许路由以树形结构排布，路由之间可以设置优先级。简单的介绍可以参考[Introducing Zend Framework 2.0 Router](http://evan.pro/zf2-router-talk.html "http://evan.pro/zf2-router-talk.html")。所以ZF2的路由可以实现分别在每个模块下设置，同时可以在某些模块提高优先级别。非常适合大规模应用的部署。

在路由启动过程中，Zend\Mvc\RouteListener→onRoute()被触发，路由从树形结构逐一匹配，最终以Zend\Mvc\Router\RouteMatch对象的形式返回一个最适配的路由。

### Dispatch分发过程

ZF2的Dispatch分发其实有两次，一次是在Zend\Mvc\Application中，目的是将匹配的RouteMatch通过参数定位到某个特定的Controller，另一次是在Zend\Mvc\Controller，目的是将Request/Response注入，同时运行对应的Action。

流程如下

```php
//分发事件被触发
Zend\Mvc\DispatchListener->onDispatch();
 
//根据匹配路由的参数定位到某个controller
$controller = $controllerLoader->get($controllerName);
 
//触发controlller的dispatch
$return   = $controller->dispatch($request, $response);
```

## 发送最终响应并结束MVC

ZendSkeletonApplication/public/index.php

```php
$response->send();
```

分发结束后，如果正确的从controller获得响应，会继续运行

```php
Zend\Mvc\Application->completeRequest()
```

这里会触发MVC事件的最后两个

1. render 渲染

2. finish 结束

```php
//调用MVC默认策略器的render事件
Zend\Mvc\View\DefaultRenderingStrategy->render();
```

Render事件会将Zend\View的各部分整合，最终组装成一个Zend\Http\PhpEnvironment\Response，发送给用户。

这就是Zend2.0的MVC完整过程。

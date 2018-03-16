---
slug: phalcon-mvc-process
date: '2014-05-30 23:29:41'
title: Phalcon Framework的Mvc结构及启动流程（部分源码分析）
---

很久没更新Blog甚是惭愧，但是[工作方面还是有不少进展](http://avnpc.com/pages/fought-side-by-side-with-me-in-wallstreetcn)，技术方面一个重大的转变是我选择了[Phalcon Framework](http://phalconphp.com/en/)作为未来一段时间的核心框架。技术选型的原因会单开一篇Blog另说，本次优先对[Phalcon的MVC架构与启动流程](http://avnpc.com/pages/phalcon-mvc-process)进行分析说明，如有遗漏还望指出。

Phalcon本身有支持创建多种形式的Web应用项目以应对不同场景，包括[迷你应用](http://docs.phalconphp.com/en/latest/reference/micro.html)、[单模块标准应用](http://docs.phalconphp.com/en/latest/reference/applications.html#single-module)、以及较复杂的[多模块应用](http://docs.phalconphp.com/en/latest/reference/applications.html#multi-module)

本次以最复杂的多模块应用为例，Phalcon版本为1.3.2，用一个Phalcon所创建的标准项目来分析

##创建项目

Phalcon环境配置安装后，可以通过命令行生成一个标准的Phalcon多模块应用

     phalcon project eva --type modules

入口文件为`public/index.php`，简化后一共5行，包含了整个Phalcon的启动流程，以下将按顺序说明

~~~ php
require __DIR__ . '/../config/services.php';
$application = new Phalcon\Mvc\Application();
$application->setDI($di);
require __DIR__ . '/../config/modules.php';
echo $application->handle()->getContent();
```

##DI注册阶段

Phalcon的所有组件服务都是通过[DI（依赖注入）](http://docs.phalconphp.com/en/latest/api/Phalcon_DI.html)进行组织的，这也是目前大部分主流框架所使用的方法。通过DI，可以灵活的控制框架中的服务：哪些需要启用，哪些不启用，组件的内部细节等等，因此Phalcon是一个松耦合可替换的框架，完全可以通过DI替换MVC中任何一个组件。

~~~ php
require __DIR__ . '/../config/services.php';
```

这个文件中默认注册了`Phalcon\Mvc\Router`（路由）、`Phalcon\Mvc\Url`（Url）、`Phalcon\Session\Adapter\Files`（Session）三个最基本的组件。同时当MVC启动后，DI中默认注册的服务还有很多，可以通过DI得到所有当前已经注册的服务：

```
$services = $application->getDI()->getServices();
foreach($services as $key => $service) {
        var_dump($key);
        var_dump(get_class($application->getDI()->get($key)));
}
```

打印看到Phalcon还注册了以下服务：

- `dispatcher` : `Phalcon\Mvc\Dispatcher` 分发服务，将路由命中的结果分发到对应的Controller
- `modelsManager` : `Phalcon\Mvc\Model\Manager` Model管理
- `modelsMetadata` : `Phalcon\Mvc\Model\MetaData\Memory`  ORM表结构
- `response` : `Phalcon\Http\Response`  响应
- `cookies` : `Phalcon\Http\Response\Cookies`  Cookies
- `request` : `Phalcon\Http\Request`  请求
- `filter` : `Phalcon\Filter` 可对用户提交数据进行过滤
- `escaper` : `Phalcon\Escaper`  转义工具
- `security` : `Phalcon\Security`  密码Hash、防止CSRF等
- `crypt` : `Phalcon\Crypt`  加密算法
- `annotations` : `Phalcon\Annotations\Adapter\Memory`  注解分析
- `flash` : `Phalcon\Flash\Direct`  提示信息输出
- `flashSession` : `Phalcon\Flash\Session` 提示信息通过Session延迟输出
- `tag` : `Phalcon\Tag` View的常用Helper



而每一个服务都可以通过DI进行替换。接下来实例化一个标准的MVC应用，然后将我们定义好的DI注入进去

```
$application = new Phalcon\Mvc\Application();
$application->setDI($di);
```



##模块注册阶段

与DI一样，Phalcon建议通过引入一个独立文件的方式注册所有需要的模块：

```
require __DIR__ . '/../config/modules.php';
```

这个文件的内容如下

```
$application->registerModules(array(
    'frontend' => array(
        'className' => 'Eva\Frontend\Module',
        'path' => __DIR__ . '/../apps/frontend/Module.php'
    )
));
```

可以看到Phalcon所谓的模块注册，其实只是告诉框架MVC模块的引导文件`Module.php`所在位置及类名是什么。


##MVC阶段

`$application->handle()`是整个MVC的核心，这个函数中处理了路由、模块、分发等MVC的全部流程，处理过程中在关键位置会通过事件驱动触发一系列`application:`事件，方便外部注入逻辑，最终返回一个`Phalcon\Http\Response`。整个[`handle`方法的过程](https://github.com/phalcon/cphalcon/blob/1.3.2/ext/mvc/application.c#L303)并不复杂，下面按顺序介绍：


### 基础检查


首先检查DI，如果没有任何DI注入，会抛出错误

> A dependency injection object is required to access internal services

然后从DI启动EventsManager，并且通过EventsManager触发事件`application:boot`

###路由阶段

接下来进入路由阶段，从DI中获得路由服务`router`，将uri传入路由并调用路由的[`handle()`方法](http://docs.phalconphp.com/en/latest/api/Phalcon_Mvc_Router.html)。

路由的handle方法负责将一个uri根据路由配置，转换为相应的Module、Controller、Action等，这一阶段接下来会检查路由是否命中了某个模块，并通过`Router->getModuleName()`获得模块名。

如果模块存在，则进入模块启动阶段，否则直接进入分发阶段。

注意到了么，在Phalcon中，**模块启动是后于路由的**，这意味着Phalcon的模块功能比较弱，我们无法在某个未启动的模块中注册全局服务，甚至无法简单的在当前模块中调用另一个未启动模块。这可能是Phalcon模块功能设计中最大的问题，解决方法暂时不在本文的讨论范围内，以后会另开文章介绍。

####模块启动

模块启动时首先会触发`application:beforeStartModule`事件。事件触发后检查模块的正确性，根据`modules.php`中定义的`className`、`path`等，将模块引导文件加载进来，并调用模块引导文件中必须存在的方法

- `Phalcon\Mvc\ModuleDefinitionInterface->registerAutoloaders ()`
- `Phalcon\Mvc\ModuleDefinitionInterface->registerServices (Phalcon\DiInterface $dependencyInjector)`

`registerAutoloaders()`用于注册模块内的命名空间实现自动加载。`registerServices ()`用于注册模块内服务，在官方示例中`registerServices ()`注册并定义了`view`服务以及模板的路径，并且注册了数据库连接服务`db`并设置数据库的连接信息。

模块启动完成后触发 `application:afterStartModule`事件，进入分发阶段

### 分发阶段（Dispatch）

分发过程由`Phalcon\Mvc\Dispatcher`（分发器）来完成，所谓分发，在Phalcon里本质上是分发器根据路由命中的结果，调用对应的Controller/Action，最终获得Action返回的结果。

分发开始前首先会准备View，虽然View理论上位于MVC的最后一环，但是如果在分发过程中出现任何问题，通常都需要将问题显示出来，因此View必须在这个环节就提前启动。Phalcon没有准备默认的View服务，需要从外部注入，在多模块demo中，View的注入官方推荐在模块启动阶段完成的。如果是单模块应用，则可以在最开始的DI阶段注入。

如果始终没有View注入，会抛出错误

> Service 'view' was not found in the dependency injection container

导致分发过程直接中断。

分发需要Dispatcher，Dispatcher同样从DI中取得。然后将router中得到的参数（NamespaceName / ModuleName / ControllerName /  ActionName / Params），全部复制到Dispatcher中。

分发开始前，会调用View的`start()`方法。具体可以参考[View相关文档](http://docs.phalconphp.com/en/latest/reference/views.html#hierarchical-rendering)，其实`Phalcon\Mvc\View->start()`就是PHP的输出缓冲函数`ob_start`的一个简单封装，分发过程中所有输出都会被暂存到缓冲区。

分发开始前还会触发事件`application:beforeHandleRequest`。

正式开始分发会调用`Phalcon\Mvc\Dispatcher->dispatch()`。

#### Dispatcher内的分发处理

进入Dispatcher后会发现Dispatcher对整个分发过程进行了进一步细分，并且在分发的过程中会按顺序触发非常多的[分发事件](http://docs.phalconphp.com/en/latest/reference/dispatching.html#dispatch-loop-events)，可以通过这些分发事件进行更加细致的流程控制。部分事件提供了可中断的机制，只要返回`false`就可以跳过Dispatcher的分发过程。

由于分发中可以使用`Phalcon\Mvc\Dispatcher->forward()`来实现Action的复用，因此分发在内部会通过循环实现，通过检测一个全局的`finished`标记来决定是否继续分发。当以下几种情况时，分发才会结束：

- Controller抛出异常
- `forward`层数达到最大（256次）
- 所有的Action调用完毕


### 渲染阶段 View Render

分发结束后会触发`application:afterHandleRequest`，接下来通过`Phalcon\Mvc\Dispatcher->getReturnedValue()`取得分发过程返回的结果并进行处理。


由于Action的逻辑在框架外，Action的返回值是无法预期的，因此这里根据返回值是否实现`Phalcon\Http\ResponseInterface`接口进行区分处理。


#### 当Action返回一个非`Phalcon\Http\ResponseInterface`类型

此时认为返回值无效，由View自己重新调度Render过程，会触发`application:viewRender`事件，同时从Dispatcher中取得ControllerName /  ActionName / Params作为`Phalcon\Mvc\View->render()`的入口参数。

Render完毕后调用`Phalcon\Mvc\View->finish()`结束缓冲区的接收。

接下来从DI获得resonse服务，将`Phalcon\Mvc\View->getContent()`获得的内容置入response。


#### 当Action返回一个`Phalcon\Http\ResponseInterface`类型


此时会将Action返回的Response作为最终的响应，不会重新构建新的Response。


### 返回响应

通过前面的流程，无论中间经历了多少分支，最终都会汇总为唯一的响应。此时会触发`application:beforeSendResponse`，并调用

- `Phalcon\Http\Response->sendHeaders()`
- `Phalcon\Http\Response->sendCookies()`

将http的头部信息先行发送。至此，`Application->handle()`对于请求的处理过程全部结束，对外返回一个`Phalcon\Http\Response`响应。

### 发送响应

HTTP头部发送后一般把响应的内容也发送出去：

```
echo $application->handle()->getContent();
```

这就是Phalcon Framework的完整MVC流程。


## 流程控制


分析MVC的启动流程，无疑是希望对流程有更好的把握和控制，方法有两种：


### 自定义启动

按照上面的流程，我们其实完全可以自己实现`$application->handle()->getContent()`这一流程，下面就是一个简单的替代方案，代码中暂时没有考虑事件的触发。

~~~ php
//Roter
$router = $di['router'];
$router->handle();

//Module handle
$modules = $application->getModules();
$routeModule = $router->getModuleName();
if (isset($modules[$routeModule])) {
    $moduleClass = new $modules[$routeModule]['className']();
    $moduleClass->registerAutoloaders();
    $moduleClass->registerServices($di);
}

//dispatch
$dispatcher = $di['dispatcher'];
$dispatcher->setModuleName($router->getModuleName());
$dispatcher->setControllerName($router->getControllerName());
$dispatcher->setActionName($router->getActionName());
$dispatcher->setParams($router->getParams());

//view
$view = $di['view'];
$view->start();
$controller = $dispatcher->dispatch();
//Not able to call render in controller or else will repeat output
$view->render(
    $dispatcher->getControllerName(),
    $dispatcher->getActionName(),
    $dispatcher->getParams()
);
$view->finish();

$response = $di['response'];
$response->setContent($view->getContent());
$response->sendHeaders();
echo $response->getContent();
```


###流程清单

为了方便查找，将整个流程整理为一个树形清单如下：

- 初始化DI (config/services.php) `$di = new FactoryDefault();`
  - 设置路由 `$di['router'] = function () {}`
  - 设置URL `$di['url'] = function () {}`
  - 设置Session `$di['session'] = function () {}`
- 初始化Application (public/index.php)
  - 实例化App `$application = new Application();`
  - 注入DI `$application->setDI($di);`
  - 注册模块 (config/modules.php) `$application->registerModules()`
- 启动Application (ext/mvc/application.c)  `$application->handle()`
  - 检查DI
  - <span class="label label-info">E</span> 触发事件  `application:boot`
  - 路由启动 `$di['router']->handle()`
    - 获得模块名 `$moduleName = $di['router']->getModuleName()`，如果没有则从 `$application->getDefaultModule`获取
  - 模块启动 (如果路由命中)
     -  <span class="label label-info">E</span> 触发事件 `application:beforeStartModule`
     - 调用模块初始化方法 (Module.php) `registerAutoloaders()` 以及 `registerServices()`
     - <span class="label label-info">E</span> 触发事件 `application:afterStartModule`
  - 分发
     - 初始化View
     - 初始化Dispatcher，将Router中的参数复制到Dispatcher
     - 调用View `View->start()`开启缓冲区
     - <span class="label label-info">E</span> 触发事件 `application:beforeHandleRequest`
     - 开始分发 (etc/dispatcher.c) `Dispatcher->dispatch()` 
         - <span class="label label-info">E</span> 触发事件 `dispatch:beforeDispatchLoop`
         - 循环开始单次分发
             - <span class="label label-info">E</span> 触发事件 `dispatch:beforeDispatch`
             - 根据Dispatcher携带的Module、Namespace、Controller、Action获得完整的类与方法名，如果找不到则触发事件 <span class="label label-info">E</span> `dispatch:beforeException`
             - <span class="label label-info">E</span> 触发事件 `dispatch:beforeExecuteRoute`
             - 调用`Controller->beforeExecuteRoute()`
             - 调用`Controller->initialize()`
             - <span class="label label-info">E</span> 触发事件 `dispatch:afterInitialize`
             - 调用Action方法
             - <span class="label label-info">E</span> 触发事件 `dispatch:afterExecuteRoute`
             - <span class="label label-info">E</span> 触发事件 `dispatch:afterDispatch`
         - Action内如果有forward()，开始下一次分发
       - <span class="label label-info">E</span> 全部分发结束，触发事件 `dispatch:afterDispatchLoop`
       - Application获得分发后的输出 `$dispatcher->getReturnedValue()`
       - <span class="label label-info">E</span> 触发事件 `application:afterHandleRequest` 分发结束
    - 渲染，Appliction如果从分发拿到`Phalcon\Http\ResponseInterface`类型的返回，则渲染直接结束
      - <span class="label label-info">E</span> 触发事件 `application:viewRender` 分发结束
      - 调用 `Phalcon\Mvc\View->render()`，入口参数为Dispatcher的 ControllerName / ActionName / Params
      - 调用 `Phalcon\Mvc\View->finish()`结束缓冲区的接收
  - 准备响应
      -  将`Phalcon\Mvc\View->getContent()`通过`Phalcon\Http\Response->setContent()`放入Response
      - <span class="label label-info">E</span> 触发事件 `application:beforeSendResponse` 
      - 调用`Phalcon\Http\Response->sendHeaders()`发送头部
      - 调用`Phalcon\Http\Response->sendCookies()`发送Cookie
      - 将准备好的响应作为`$application->handle()`的返回值返回
- 发送响应
   - `echo $application->handle()->getContent();`


###MVC事件

Phalcon作为C扩展型的框架，其优势就在于高性能，虽然我们可以通过上一种方法自己实现整个启动，但更好的方式仍然是避免替换框架本身的内容，而使用事件驱动。

下面梳理了整个MVC流程中所涉及的可被监听的事件，可以根据不同需求选择对应事件作为切入点：

- DI注入
  -  `application:boot`  应用启动
- 路由阶段
  -  模块启动
    - `application:beforeStartModule` 模块启动前
    - `application:afterStartModule` 模块启动后
- 分发阶段
  - `application:beforeHandleRequest`进入分发器前
  - 开始分发
     - `dispatch:beforeDispatchLoop` 分发循环开始前
     - `dispatch:beforeDispatch`  单次分发开始前
     - `dispatch:beforeExecuteRoute`  Action执行前
     - `dispatch:afterExecuteRoute`  Action执行后
     - `dispatch:beforeNotFoundAction`  找不到Action
     - `dispatch:beforeException`  抛出异常前
     - `dispatch:afterDispatch` 单次分发结束
     - `dispatch:afterDispatchLoop` 分发循环结束
  - `application:afterHandleRequest` 分发结束
- 渲染阶段
   - `application:viewRender` 渲染开始前
- 发送响应
   - `application:beforeSendResponse`  最终响应发送前



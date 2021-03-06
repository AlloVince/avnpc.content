---
published: true
date: '2012-10-29 22:32:18'
tags:
  - ZF2
  - Zend Framework 2
  - EventManager
  - 事件驱动
  - MVC
author: AlloVince
title: 在 ZF2 中实现 Zend Framework 的 Controller init/preDispatch/postDispatch 方法
---

在 Zend Framework 1 中，Controller 里约定了一些默认的方法来实现钩子，包括

 - Controller 初始化时调用的 init()方法
 - 派遣前的方法 preDispatch()
 - 派遣后的方法 postDispatch()

在 Zend Framework 2 中，Controller 不再提供这些方法，而改为更灵活的事件驱动，不过如果还想实现类似功能要怎么办呢？这里给出[ZF2 中实现 init/preDispatch/postDispatch 的实现方法](http://avnpc.com/pages/controller-init-predispath-postdispath-in-zf2)，对比一下吧：


可能有童鞋会想到 php 提供了构造函数这一功能，所以我们完全可以这样来处理一些需要在 controller 启动时初始化的数据：

```php
use Zend\Mvc\Controller\AbstractActionController;
class MyController extends AbstractActionController
{
    public function __construct()
    {
    }
}
```

但是在构造函数启动时，ZF2 还没有将外部的 ServiceLocator 注入进来，所以在构造函数__construct()中，我们是无法使用 ZF2 内建的一些方法与服务的，因此我们需要将初始化的时机稍作调整，就可以实现 ZF1 中的 init 方法了：

```php
use Zend\Mvc\Controller\AbstractActionController;
use Zend\EventManager\EventManagerInterface;
class MyController extends AbstractActionController
{

    public function setEventManager(EventManagerInterface $events)
    {
        parent::setEventManager($events);
        $this->init();
    }

    public function init()
    {
    }
}
```

setEventManager 在 controller 启动时，ZF2 将外部的 EventManager 注入到当前 controller 时调用，在 setEventManager 可以模拟 ZF1 的 init 方法。

看起来比 ZF1 更罗嗦了是吗。那是因为我们完全忘掉了事件驱动这个法宝。ZF2 中与与 Controller 相关的 MVC 事件有两个：MvcEvent::EVENT_DISPATCH 和 MvcEvent::EVENT_RENDER，忘掉的话可能需要复习一下[ZF2 默认的 Mvc 事件](http://avnpc.com/pages/zf2-mvc-process)。

理论上来讲，在 ZF2 中，我们可以在任意模块，任意位置（Config/Module/Controller/Model）对任意 Controller 的逻辑进行修改。

比如说，我在原本的系统中加入了用户模块，现在想要把当前登录用户的信息放入**所有**的模板中。如果是 ZF1，这可能会产生不小的工作量，但是在 ZF2 中，我们只需要在用户模块的 Module.php 中加入一个方法：

```php
namespace User;
use Zend\Mvc\MvcEvent;

class Module
{
    public function onBootstrap($e)
    {
        $app = $e->getParam('application');
        $app->getEventManager()->attach(MvcEvent::EVENT_RENDER, array($this, 'setUserToView'), 100);
    }

    public function setUserToView($event)
    {
        $user = array();  //login userinfo here
        $viewModel = $event->getViewModel();
        $viewModel->setVariables(array(
            'user' => $user,
        ));
    }
}
```

清爽、简单，假如以后要移除用户信息，也只需要修改 User\Module 一处。

同理我们可以在 Controller 中模拟以前的 preDispatch/postDispatch 方法：

```php
use Zend\Mvc\Controller\AbstractActionController;
class MyController extends AbstractActionController
{
    protected function attachDefaultListeners()
    {
        parent::attachDefaultListeners();
        $events = $this->getEventManager();
        $events->attach(MvcEvent::EVENT_DISPATCH, array($this, 'preDispatch'), 100);
        $events->attach(MvcEvent::EVENT_DISPATCH, array($this, 'postDispatch'), -100);
    }

    public function preDispatch()
    {
    }

    public function postDispatch()
    {

    }

}
```

在 MVC 的 DISPATCH 事件上注册新的方法，并调整优先级，就是这么简单。

Dispacth 事件有点特殊，参考[ZF2 的 Mvc 流程](http://avnpc.com/pages/zf2-mvc-process)，实际会被触发两次，以下的测试代码应该可以帮助理解事件的顺序：


```php
class Module
{
    public function onBootstrap($e)
    {
        $eventManager = $e->getApplication()->getEventManager();
        $eventManager->attach(MvcEvent::EVENT_DISPATCH, array($this, 'onApplicationPreDispacth'), 100);
        $eventManager->attach(MvcEvent::EVENT_DISPATCH, array($this, 'onApplicationPostDispacth'), -100);

    }

    public function onApplicationPreDispacth($event)
    {
        echo 1;
    }

    public function onApplicationPostDispacth($event)
    {
        echo 6;
    }
}

class MyController extends AbstractActionController
{
    public function indexAction()
    {
    }

    public function setEventManager(EventManagerInterface $events)
    {
        parent::setEventManager($events);
        $this->init();
    }

    protected function attachDefaultListeners()
    {
        parent::attachDefaultListeners();
        $events = $this->getEventManager();
        $events->attach(MvcEvent::EVENT_DISPATCH, array($this, 'preDispatch'), 100);
        $events->attach(MvcEvent::EVENT_DISPATCH, array($this, 'postDispatch'), -100);
        $eventManager = $this->getServiceLocator()->get('Application')->getEventManager();
        $eventManager->attach(MvcEvent::EVENT_RENDER, array($this, 'preRender'), 100);
        $eventManager->attach(MvcEvent::EVENT_RENDER, array($this, 'postRender'), -100);
    }

    public function preDispatch()
    {
        echo 4;
    }

    public function postDispatch()
    {
        echo 5;
    }

    public function preRender()
    {
        echo 7;
    }

    public function postRender()
    {
        echo 8;
    }

    public function init()
    {
        echo 3;
    }

    public function __construct()
    {
        echo 2;
    }
}
```


最后的输出为 12345678。



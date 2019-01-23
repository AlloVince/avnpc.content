---
published: true
date: '2012-07-26 21:01:02'
tags:
  - ZF2
  - Zend Framework 2
  - EventManager
  - 事件驱动
author: AlloVince
title: ZF2 小 TIP：使用事件驱动为模块快速设置模板
---

在 ZF1 中，对一部分页面设置一个不同的 Layout 可能需要在每一个 Controller 中单独设置。在 ZF2 中，事件驱动的支持让 Layout 的设置变得非常灵活。

比如要对 Admin 模块单独设置一个 admin 模板，只需要短短 5 行代码

```php
<?php
namespace Admin;
 
use Zend\ModuleManager\ModuleManager;
 
class Module 
{
    public function init(ModuleManager $moduleManager)
    {
        $sharedEvents = $moduleManager->getEventManager()->getSharedManager();
        $sharedEvents->attach(__NAMESPACE__, 'dispatch', function($e) {
            $controller = $e->getTarget();
            $controller->layout('layout/admin');
        }, 100);
    }
}
```

上例中，对 MVC 的 Dispath 分发事件绑定了一个闭包，闭包中切换 controller 的 Layout 为 Admin。同样的道理，可以通过事件驱动很简单的实现 View 根目录切换等原本非常繁琐的工作。只是这一切需要对[ZF2 的 MVC 启动流程](/pages/zf2-mvc-process)有所了解。

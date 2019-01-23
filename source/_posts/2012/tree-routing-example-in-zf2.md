---
published: true
date: '2012-11-05 18:45:40'
tags:
  - ZF2
  - Zend Framework 2
  - Router
  - 路由
author: AlloVince
title: ZF2多级树形路由Route配置实例
---

Zend Framework 2的路由设置非常灵活，每一个路由都可以呈现树形结构，同时还可以为一个路由及其子路由设置不同的Controller与Action。在此之上又提供了非常丰富的路由种类，查阅[ZF2路由相关文档](http://framework.zend.com/manual/2.0/en/modules/zend.mvc.routing.html)可以了解到路由的常用方法。

虽然可供选择的余地很大，不过在实际使用中，Zend\Mvc\Router\Http\Segment应该是适用范围最广的，推荐使用。因为在实际项目中，经常需要处理URL末尾的斜线/问题。比如 http://avnpc.com/life 与 http://avnpc.com/life/ 应该指向同一处，此时如果用Zend\Mvc\Router\Http\Literal，就需要对有斜线和无斜线两种情况分别写路由：

```php
'router' => array(
    'routes' => array(
        'life' => array(
            'type' => 'literal',
            'options' => array(
                'route'    => '/life',
                'defaults' => array(
                    'controller' => 'LifeController',
                    'action'     => 'index',
                ),
            ),
        ),
        'lifeslash' => array(
            'type' => 'literal',
            'options' => array(
                'route'    => '/life/',
                'defaults' => array(
                    'controller' => 'LifeController',
                    'action'     => 'index',
                ),
            ),
        ),
    ),
),
```

使用Segment路由则只需配置一次：

```php
'router' => array(
    'routes' => array(
        'life' => array(
            'type' => 'segment',
            'options' => array(
                'route'    => '/life[/]',
                'defaults' => array(
                    'controller' => 'LifeController',
                    'action'     => 'index',
                ),
            ),
        ),
    ),
),
```


那么下面用一个比较复杂的实例来说明[如何在ZF2中配置一个多级树形路由Route](http://avnpc.com/pages/tree-routing-example-in-zf2)。举例一个在微博或SNS系统中都比较常见的应用场景，我们希望项目的URL中：

 - /user 对应用户列表页面
 - /user/:user_id 对应用户的个人主页，比如 /user/AlloVince 就对应AlloVince用户的个人主页
 - /user/:user_id/blog/ 对应用户的博客列表页面，比如/user/AlloVince/blog 就会列出AlloVince写过的Blog
 - /user/:user_id/blog/:blog_id 对应用户的一篇博客文章

那么最终写成Route应该是这样的：

```php
'router' => array(
    'routes' => array(
        'user' => array(
            'type' => 'Segment',
            'options' => array(
                'route' => '/user[/]',
                'defaults' => array(
                    'controller' => 'UserController',
                    'action' => 'index',
                ),
            ),
            'may_terminate' => true,
            'child_routes' => array(
                'profile' => array(
                    'type' => 'Segment',
                    'options' => array(
                        'route' => '[:id][/]',
                        'constraints' => array(
                            'id' => '[a-zA-Z0-9_-]+'
                        ),
                        'defaults' => array(
                            'action' => 'get'
                        ),
                    ),
                    'may_terminate' => true,
                    'child_routes' => array(
                        'blog' => array(
                            'type' => 'Segment',
                            'options' => array(
                                'route' => 'blog[/]',
                                'constraints' => array(
                                ),
                                'defaults' => array(
                                    'action' => 'blog'
                                )
                            ),
                            'may_terminate' => true,
                            'child_routes' => array(
                                'post' => array(
                                    'type' => 'Segment',
                                    'options' => array(
                                        'route' => '[:post_id][/]',
                                        'constraints' => array(
                                            'post_id' => '[a-zA-Z0-9_-]+'
                                        ),
                                        'defaults' => array(
                                            'action' => 'post'
                                        )
                                    ),
                                    'may_terminate' => true,
                                ),
                            ),
                        ),
                    ), //profile child_routes end
                ), //profile end
            ), //user child_routes end
        ), //user end
    ),
),
```

需要注意的是may_terminate这个参数，代表路由是否可以被终止，如果没有这个参数的话，路由在匹配到父级URL时会继续向下匹配子路由。比如我们去掉上例的第一个may_terminate。然后访问/user。则会继续进入到profile子路由然后抛出错误。




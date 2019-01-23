---
published: true
date: '2012-11-14 23:57:50'
tags:
  - ZF2
  - Zend Framework 2
  - Email
  - DI
author: AlloVince
title: 使用 ZF2 的 DI 操作 Zend\Mail 发送邮件
---

Zend Framework 2 完整的实现了[DI](http://framework.zend.com/manual/2.0/en/modules/zend.di.introduction.html)，也就是依赖注入功能，但是在正式发行的 ZF2 中，DI 已经基本被 ServiceManager 所取代，一个 ZF2 项目几乎可以不接触 DI 这一层。

但是 DI 仍然是 ZF2 中非常富有魅力的一个组件，通过一个的[DI 操作 Zend\Mail 发送邮件](http://avnpc.com/pages/zend-mail-usage-by-di-in-zf2)的实例来了解一下 DI 吧。

正常的 Zend\Mail 使用中，我们一般需要实例化一个 Zend\Mail\Message 加一个 Zend\Mail\Transport。然后将 Message 置入 Transport 进行发送。详细的用法 ZF2 手册的[Zend\Mail](http://framework.zend.com/manual/2.0/en/modules/zend.mail.introduction.html)一节写的比较完整，不再赘述。

同样的功能如果使用 DI，就变成了下面这样：

```php
use Zend\Mail\Message;
use Zend\Mail\Transport;
use Zend\Di\Di;
use Zend\Di\Config as DiConfig;

$diConfig = array('instance' => array(
    'Zend\Mail\Transport\FileOptions' => array(
        'parameters' => array(
            'path' => __DIR__,
        )
    ),
    'Zend\Mail\Transport\File' => array(
        'injections' => array(
            'Zend\Mail\Transport\FileOptions'
        )
    ),
    'Zend\Mail\Transport\SmtpOptions' => array(
        'parameters' => array(
            'name'              => 'sendgrid',
            'host'              => 'smtp.sendgrid.net',
            'port' => 25,
            'connectionClass'  => 'login',
            'connectionConfig' => array(
                'username' => 'username',
                'password' => 'password',
            ),
        )
    ),
    'Zend\Mail\Message' => array(
        'parameters' => array(
            'headers' => 'Zend\Mail\Headers',
            'Zend\Mail\Message::setTo:emailOrAddressList' => 'allo.vince@gmail.com',
            'Zend\Mail\Message::setTo:name' => 'AlloVince',
            'Zend\Mail\Message::setFrom:emailOrAddressList' => 'info@evaengine.com',
            'Zend\Mail\Message::setFrom:name' => 'EvaEngine',
        )
    ),
    'Zend\Mail\Transport\Smtp' => array(
        'injections' => array(
            'Zend\Mail\Transport\SmtpOptions'
        )
    ),
));

$di = new Di();
$di->configure(new DiConfig($diConfig));

$transport = $di->get('Zend\Mail\Transport\Smtp');
$message = $di->get('Zend\Mail\Message');
$message->setSubject("Mail Subject")->setBody('Mail Content');
$transport->send($message);
```

我们通过一个庞大的数组，配置了 Zend\Mail 初始化所需要的所有细节。包括两种 Transport\File 的保存路径，SMTP 的服务器/用户名/密码/端口，默认的收件人及发件人等等。所以可以看到最终通过 DI 实例化 Transport 和 Message 时，用法是极简的，无需再指定发件人/收件人/邮件服务器等，因为所有的细节都已经封装在 DI 的配置里了。

例子中的邮件 SMTP 使用了 Sendgrid 的服务，你需要填入自己的帐号和密码，同样你可以使用 Gmail 或其他邮箱作为发信服务。

如果想使用其他 Transport 发信，只需要简单更改为：

```php
$transport = $di->get('Zend\Mail\Transport\File');
```

运行就会在当前目录下生成邮件 Log。

这样与普通的实例化操作相比有什么优缺点呢？个人理解为：

1. DI 可以通过一个配置文件控制程序的所有细节，而配置文件是非常容易共享和修改的。只要将 DI 放入全局配置文件，可以在整个项目中避免大量重复代码，实现程序的复用。
2. DI 虽然内部逻辑复杂，但是在外部看来非常简单：放入一个配置，即得到一个结果，非常容易做单元测试。
3. 无需在程序内部做一些参数有无的判断，只需要保证外部配置文件的完整即可。

缺点也很明显：

1. 不直观，容易陷入[元数据式编程](http://en.wikipedia.org/wiki/Metaprogramming)的泥潭，可能一个 DI 实现的项目，修改配置文件的时间要远远大于编程的时间。
2. 难以调试，DI 就像黑盒，我们只能清楚的知道入口与出口的情况，至于内部发生了什么则很难得知。一个 DI 的编写往往要消耗更多时间。

所以是否在项目中使用 DI，就需要自己权衡了。



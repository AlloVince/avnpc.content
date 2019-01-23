---
published: true
date: '2012-11-21 14:52:29'
tags:
  - ZF2
  - Zend Framework 2
  - Email
  - DI
author: AlloVince
title: Zend\Mail进阶：在ZF2的邮件中使用模板、多个附件以及用DI整合
---

在解决了基础的[ZF2使用DI操作Zend\Mail发送邮件](http://avnpc.com/pages/zend-mail-usage-by-di-in-zf2)之后，可能我们关注的问题会转移到更加实际的方面，包括：

- [如何为ZF2中的Zend\Mail设置模板并可以在模板中使用ViewHelper](http://avnpc.com/pages/zf2-mail-advanced-usage-for-template-multi-attachment#template)
- [如何在ZF2中发送带有附件(Attachment)的邮件](http://avnpc.com/pages/zf2-mail-advanced-usage-for-template-multi-attachment#attachment)
- [如何更好地使用DI来完成这一切](http://avnpc.com/pages/zf2-mail-advanced-usage-for-template-multi-attachment#di)

在Zend\Mail中使用模板
--------------------

首先想到的自然是使用Zend\View作为邮件的模板引擎，这样不但可以使用Zend\View的所有Helper，稍加改进还可以传入整个系统的ServiceLocator让应用范围更广。

那么在庞大的Zend\View中，哪些部分可以构成一个最简的模板引擎呢，我在[Zend Framework 2.0的Mvc结构及启动流程分析](http://avnpc.com/pages/zf2-mvc-process)有过对Zend\View的简单介绍，这里直接说结论，要构建一个最简的ZF2模板引擎，需要：

1. 一个View\Renderer 渲染器，这里我们选用PhpRenderer，如果你需要整合其他第三方引擎，这里需要自己写对应的View\Renderer
2. 一个View\Resolver 决策器，因为一个渲染器必须对应一个决策器
3. 一个ViewModel，用于向模板中放置变量，当然如果你的模板没有变量替换的需求，ViewModel就不是必需的

以上3个部分就可以建立起一个最简的ZF2模板引擎。

在决策器的选择上，ZF2默认的Resolver有三种：

- Resolver\TemplateMapResolver 以键值对形式指定模板路径，相当于硬编码
- Resolver\TemplatePathStack 指定若干个目录，然后寻找目录下最可能匹配的模板
- Resolver\AggregateResolver 其实只是一个队列容器，可以同时支持上述两种Resolver以及用户自定义的Resolver

所以如果每个邮件对应的模板路径都不同，可以直接采用TemplateMapResolver。如果很多模板存放在同一目录下，那么使用TemplatePathStack会更适合，更复杂情况就需要采用AggregateResolver。

以TemplateMapResolver为例，Message与Transport则继续使用[ZF2使用DI操作Zend\Mail发送邮件](http://avnpc.com/pages/zend-mail-usage-by-di-in-zf2)文中的部分：

```php
$view = new Zend\View\Renderer\PhpRenderer();
$resolver = new Zend\View\Resolver\TemplateMapResolver();
$resolver->setMap(array(
    'mailTemplate' => __DIR__ . '/mail/template.phtml'
));
$view->setResolver($resolver);
$viewModel = new Zend\View\Model\ViewModel();
$viewModel->setTemplate('mailTemplate')
->setVariables(array(
    'user' => 'AlloVince'
));

$message->setSubject("Zend Mail with Template")
->setBody($view->render($viewModel));
$transport->send($message);
```

在同一目录下放置模板文件mail/template.phtml。模板的内容为

```
    User : <?=$this->user?> | Url : <?=$this->serverUrl()?>
```

可以简单的测试变量置入以及Helper的调用。

最后生成邮件：

```
    From: EvaEngine <info@evaengine.com>
	To: EvaEngine <allo.vince@gmail.com>
	Subject: Zend Mail with Template
	User : AlloVince | Url : http://zf2.local
```

同理如果采用TemplatePathStack，对应的地方修改为

```php
$resolver = new Zend\View\Resolver\TemplatePathStack();
$resolver->setPaths(array(
    'mailTemplate' => __DIR__
));
$viewModel->setTemplate('mail/template');
```


使用Zend\Mail发送附件 
--------------------

与Zend Framework 1最大的不同，可能就是ZF2中移除了添加附件的方法，改为自己构建一个Zend\Mime\Message然后传入Zend\Mail\Message。笔者以为这个改动不够友好，不过好在[MIME规范](http://en.wikipedia.org/wiki/MIME)整体并不复杂。而通过MIME构建邮件内容更加灵活一些。

ZF2对于附件的默认编码是8bit，实测发现8bit编码的附件无法被正确解码，一些图片作为附件后无法正确显示。这个BUG暂时没有时间深究，希望有朋友能告知缘由。

比较保险的做法是对附件采用base64编码，参考下例，附件为同目录下的attachment.jpg：

```php
use Zend\Mime\Message as MimeMessage;
use Zend\Mime\Part;

$mimeMessage = new MimeMessage();
$messageText = new Part('Mail Content');
$messageText->type = 'text/html';

$data = fopen('attachment.jpg', 'r');
$messageAttachment = new Part($data);
$messageAttachment->type = 'image/jpg';
$messageAttachment->filename = 'attachment.jpg';
$messageAttachment->encoding = Zend\Mime\Mime::ENCODING_BASE64;
$messageAttachment->disposition = Zend\Mime\Mime::DISPOSITION_ATTACHMENT;

$mimeMessage->setParts(array(
    $messageText,
    $messageAttachment,
));

$message->setSubject("Mail Subject with Attachment")
        ->setBody($mimeMessage);
$transport->send($message);
```

发送的邮件为

```
From: =?utf-8?Q?EvaEngine?= <info@evaengine.com>
To: =?utf-8?Q?EvaEngine?= <allo.vince@gmail.com>
Subject: =?utf-8?Q?Mail=20Subject=20with=20Attachment?=
MIME-Version: 1.0
Content-Type: multipart/mixed;
 boundary="=_94a850a623ed49ae30132a639054f500"

This is a message in Mime Format.  If you see this, your mail reader does not support this format.

--=_94a850a623ed49ae30132a639054f500
Content-Type: text/html
Content-Transfer-Encoding: 8bit

Mail Content
--=_94a850a623ed49ae30132a639054f500
Content-Type: image/jpg
Content-Transfer-Encoding: base64
Content-Disposition: attachment; filename="attachment"

base64code here

--=_94a850a623ed49ae30132a639054f500
```


###多个附件与中文文件名

在Windows下，处理文件名中含中文字符的文件需要转换一次编码，在上例中增加：

```php
$file = iconv("UTF-8", "gb2312", '中文文件名.txt');
$data = fopen($file, 'r');
$messageTextAttachment = new Part($data);
$messageTextAttachment->filename = '中文文件名.txt';
$messageTextAttachment->encoding = Zend\Mime\Mime::ENCODING_BASE64;
$messageTextAttachment->disposition = Zend\Mime\Mime::DISPOSITION_ATTACHMENT;

$mimeMessage->setParts(array(
    $messageText,
    $messageAttachment,
    $messageTextAttachment,
));
```

就可以同时发送两个附件了。

用DI整合ZF2的邮件模板与附件发送 
--------------------------------

所有的功能，最终可以用DI整合，方便在系统中复用，有特殊需求的地方复写DI配置文件即可。下面是一个最终完整的例子：


```php
use Zend\Mail\Message;
use Zend\Mail\Transport;
use Zend\Di\Di;
use Zend\Di\Config as DiConfig;
use Zend\Mime\Message as MimeMessage;
use Zend\Mime\Part;

$diConfig = array('instance' => array(
    'Zend\View\Resolver\TemplatePathStack' => array(
        'parameters' => array(
            'paths'  => array(
                'mailTemplate' => __DIR__ . '/',
            ),
        ),
    ),
    'Zend\View\Renderer\PhpRenderer' => array(
        'parameters' => array(
            'resolver' => 'Zend\View\Resolver\TemplatePathStack',
        ),
    ),
    'Zend\View\Model\ViewModel' => array(
        'parameters' => array(
            'template' => 'mail/template',
        ),
    ),
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
            'Zend\Mail\Message::setTo:name' => 'EvaEngine',
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
$view = $di->get('Zend\View\Renderer\PhpRenderer');
$viewModel = $di->get('Zend\View\Model\ViewModel');
$message = $di->get('Zend\Mail\Message');

$viewModel->setVariables(array(
    'user' => 'AlloVince'
));

$mimeMessage = new MimeMessage();
$messageText = new Part($view->render($viewModel));
$messageText->type = 'text/plain';

$messageAttachment = new Part(fopen('attachment.jpg', 'r'));
$messageAttachment->type = 'image/jpg';
$messageAttachment->filename = 'attachment.jpg';
$messageAttachment->encoding = Zend\Mime\Mime::ENCODING_BASE64;
$messageAttachment->disposition = Zend\Mime\Mime::DISPOSITION_ATTACHMENT;

$mimeMessage->setParts(array(
    $messageText,
    $messageAttachment,
));


$message->setSubject("Mail Subject")
->setBody($mimeMessage);
$transport->send($message);
```

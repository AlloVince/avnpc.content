---
slug: zend-mail-usage-by-di-in-zf2
date: '2012-11-14 23:57:50'
title: 使用ZF2的DI操作Zend\Mail发送邮件
id: 167
tags:
  - ZF2
  - Zend Framework 2
  - Email
  - DI
---

Zend Framework 2完整的实现了[DI](http://framework.zend.com/manual/2.0/en/modules/zend.di.introduction.html)，也就是依赖注入功能，但是在正式发行的ZF2中，DI已经基本被ServiceManager所取代，一个ZF2项目几乎可以不接触DI这一层。

但是DI仍然是ZF2中非常富有魅力的一个组件，通过一个的[DI操作Zend\Mail发送邮件](http://avnpc.com/pages/zend-mail-usage-by-di-in-zf2)的实例来了解一下DI吧。

正常的Zend\Mail使用中，我们一般需要实例化一个Zend\Mail\Message加一个Zend\Mail\Transport。然后将Message置入Transport进行发送。详细的用法ZF2手册的[Zend\Mail](http://framework.zend.com/manual/2.0/en/modules/zend.mail.introduction.html)一节写的比较完整，不再赘述。

同样的功能如果使用DI，就变成了下面这样：

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

我们通过一个庞大的数组，配置了Zend\Mail初始化所需要的所有细节。包括两种Transport\File的保存路径，SMTP的服务器/用户名/密码/端口，默认的收件人及发件人等等。所以可以看到最终通过DI实例化Transport和Message时，用法是极简的，无需再指定发件人/收件人/邮件服务器等，因为所有的细节都已经封装在DI的配置里了。

例子中的邮件SMTP使用了Sendgrid的服务，你需要填入自己的帐号和密码，同样你可以使用Gmail或其他邮箱作为发信服务。

如果想使用其他Transport发信，只需要简单更改为：

    $transport = $di->get('Zend\Mail\Transport\File');

运行就会在当前目录下生成邮件Log。

这样与普通的实例化操作相比有什么优缺点呢？个人理解为：

1. DI可以通过一个配置文件控制程序的所有细节，而配置文件是非常容易共享和修改的。只要将DI放入全局配置文件，可以在整个项目中避免大量重复代码，实现程序的复用。
2. DI虽然内部逻辑复杂，但是在外部看来非常简单：放入一个配置，即得到一个结果，非常容易做单元测试。
3. 无需在程序内部做一些参数有无的判断，只需要保证外部配置文件的完整即可。

缺点也很明显：

1. 不直观，容易陷入[元数据式编程](http://en.wikipedia.org/wiki/Metaprogramming)的泥潭，可能一个DI实现的项目，修改配置文件的时间要远远大于编程的时间。
2. 难以调试，DI就像黑盒，我们只能清楚的知道入口与出口的情况，至于内部发生了什么则很难得知。一个DI的编写往往要消耗更多时间。

所以是否在项目中使用DI，就需要自己权衡了。



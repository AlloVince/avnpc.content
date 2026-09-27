---
title: "OpenID with PHP 上（with OpenID Enabled）"
date: "2009-02-06 11:58:10"
slug: "openid_with_php_and_CakePHP_1"
published: true
author: "AlloVince"
legacy_id: 125
comment_status: "closed"
comments: false
tags:
  - "AVBlog"
  - "openid"
  - "php"
  - "YD的程序员葛阁"
---
**目录** [折叠]

1. [打酱油先](#toc_96047100)
2. [准备工作](#toc_96060700)
3. [运行Demo](#toc_96072800)
4. [数据库模式运行](#toc_96089200)

AVBlog2.0开发手记之一

#### 打酱油先

[OpenID](http://openid.net/)已经不算是新鲜事物了，不过至今仍然是处于叫好不叫座的尴尬状态。推广的策略是一方面，从用户体验的角度来看，登录过程中会有可能跳转到OpenID网站，这是对已经习惯了普通注册登录流程的用户一个极大的考验。所以像[Facebook Connect](http://developers.facebook.com/connect.php)这样更简洁的应用反而更容易获得市场。也许OpenID想要获得广泛应用还需要不少改进之处，但OpenID的理念，毫无疑问会在未来大放异彩。

不过这些和本次的Blog并无太大关系，我只是出来打酱油的:P

想要让自己的网站支持OpenID，门槛并不算高，以下以PHP为例，记述PHP环境下OpenID登录过程的开发。

#### 准备工作

OpenID的通讯过程已经有很多库封装好了，在[OpenID Wiki](http://wiki.openid.net/Libraries)里可以看到不少PHP类库，本次我们选择JanRain的[OpenID Enabled](http://openidenabled.com/)。

最新的版本是Version 2.1.2，基于OpenID2.0协议，下载到本地后直接运行examples文件夹里的detect.php，可以根据提示调整开发环境，最低限度需要PHP的[PEAR](http://pear.php.net/)支持。具体情况可以根据提示灵活处理，环境调整的工作这里一笔带过。

#### 运行Demo

客户端登录，已经有现成的Demo可供参考，运行examples/consumer/index.php，输入我们想要验证的OpenID然后提交，Win32环境下会很不幸的出现无法创建/tmp/\_php_consumer_test的错误。这是为了保存登录过程中的临时文件，修改examples/consumer/common.php第53行，改为相对目录。

```php
    $store_path = "./_php_consumer_test";
```

重新运行Demo，仍然报错

> Fatal error: Define Auth_OpenID_RAND_SOURCE as null to continue with an insecure random number generator.

在Auth\OpenID\CryptUtil.php的第23行，这里定义了Auth_OpenID_RAND_SOURCE会通过一个特定文件用来生成随机数，这里可以指定本机的一个文件，也可以直接定义为null

```php
define('Auth_OpenID_RAND_SOURCE', null);
```

至此如果运行环境没有问题的话，客户端登录的Demo已经可以成功运行了。

#### 数据库模式运行

上面的例子采用了文件的方式保存登录过程中的临时文件，OpenID Enabled同时也提供了数据库方式的存储，数据库连接部分是基于PEAR的DB类实现。以Mysql为例，修改examples/consumer/index.php的getStore函数如下

```php
function &getStore() {
    require_once "Auth/OpenID/MySQLStore.php";
	$db =& DB::connect(array(
		'phptype'  => 'mysql',
		'username' => 'root',
		'password' => 'root',
		'hostspec' => 'localhost',
		'database' => 'blog'
	));
    $store = new Auth_OpenID_MySQLStore($db);
    $store->createTables();
    return $store;
}
```

变量$db指定了数据库链接信息，运行后会发现数据库里自动建立了两个表  
oid_associations和oid_nonces，用于存储OpenId登录信息。

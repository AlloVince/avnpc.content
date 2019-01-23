---
published: true
date: '2012-11-30 01:03:49'
tags:
  - ZF2
  - Zend Framework 2
  - OAuth
  - EvaEngine
  - Project
author: AlloVince
title: EvaOAuth0.1文档
---

### EvaOAuth1.0已经发布，请去[新版本页面](http://avnpc.com/pages/evaoauth)查看内容，本页内容不再维护


[EvaOAuth](http://avnpc.com/pages/evaoauth)是一个统一接口设计，兼容OAuth1.0与OAuth2.0规范的php oauth登录模块，目前支持超过20个主流网站的OAuth登录，包括：

- 国外
  1. Facebook OAuth2
  2. Twitter OAuth1
  3. Google OAuth2
  4. Github OAuth2
  5. Msn Live OAuth2
  6. Flickr OAuth1
  7. LinkedIn OAuth1
  8. Yahoo OAuth1
  9. Dropbox OAuth1
  10. Foursquare OAuth2
  11. Disqus OAuth2
- 国内
  1. 豆瓣 Douban OAuth1
  2. 豆瓣 Douban OAuth2
  3. 微博 Weibo OAuth2
  4. 人人网 Renren OAuth2
  5. 腾讯QQ Tencent OAuth2
  6. 开心网 Kaixin OAuth2
  7. 百度 Baidu OAuth2
  8. 360 Qihoo OAuth2
  9. 网易微博 Netease OAuth2
  10. 搜狐微博 Sohu OAuth1
  11. 天涯 Tianya OAuth1

EvaOAuth统一接口规范，上面的任何一个第三方网站，在使用EvaOAuth时的代码与流程都是完全一致的，也可以很简单的扩展并加入新的第三方网站。

最终可以用20行左右代码实现以上所有支持网站的完整OAuth登录授权。


获得代码
-------------

推荐在项目中使用Composer进行一键安装。

请编辑composer.json，加入

```json
    "require": {
        "AlloVince/EvaOAuth": "dev-master"
    },
```

然后运行

```shell
    php composer.phar install
```

即可。

EvaOAuth要求PHP版本必须高于5.3.3，并主要依赖以下几个ZF2模块：

- [ZendOAuth](https://github.com/zendframework/ZendOAuth)
- Zend\Session 储存Token信息，也可以自己扩展Storage\StorageInterface实现其他存储方式
- Zend\Json

在同目录下会自动创建vendor目录并下载所有的依赖，在你的项目中，只需要包含自动生成的vendor/autoload.php即可。

或者请访问[EvaOAuth的Github项目主页](https://github.com/AlloVince/EvaOAuth)。

###在Windows环境下composer.phar的安装配置

参考之前的[ZF2在Windows下的环境搭建](http://avnpc.com/pages/zend-framework-2-installation-for-windows)，假设我们的php.exe目录在d:\xampp\php，那么首先将php目录加入windows环境变量。

```
    cd d:\xampp\php
    php -r "eval('?>'.file_get_contents('https://getcomposer.org/installer'));"
```


同目录下编辑文件 composer.bat，内容为

```
    @ECHO OFF
    SET composerScript=composer.phar
    php "%~dp0%composerScript%" %*
```

运行

```
    composer -V
```
检查composer安装是否成功。

进入EvaOAuth目录下运行：

```
    php D:\xampp\php\composer.phar install
```


申请应用
-------------

实现OAuth登录必须先在相应的第三方网站上申请应用并获得的consumer key与consumer secret，每个网站可能叫法不太一样，以豆瓣为例：

访问[豆瓣开发者](http://developers.douban.com/)，我的应用->创建新应用。创建完毕后

- 豆瓣应用的API Key对应EvaOAuth的consumer key
- 豆瓣应用的Secret对应EvaOAuth的consumer secret

快速开始
-------------

假设我们将EvaOAuth文件夹命名为OAuth并可以用http://localhost/OAuth/访问，同时已经安装好了所有的依赖。我们以豆瓣的OAuth2.0为例（因为豆瓣没有限制CallbackUrl的域，非常方便测试），用几十行代码构建一个完整的OAuth登录：

###获得Request Token并跳转到第三方进行授权

首先编写一个文件request.php，内容如下：

```php
require_once './vendor/autoload.php';
use EvaOAuth\Service as OAuthService;

$oauth = new OAuthService();
$oauth->setOptions(array(
    'callbackUrl' => 'http://localhost/EvaOAuth/examples/access.php',
    'consumerKey' => 'XXX',
    'consumerSecret' => 'YYY',
));
$oauth->initAdapter('Douban', 'OAuth2');

$requestToken = $oauth->getAdapter()->getRequestToken();
$oauth->getStorage()->saveRequestToken($requestToken);
$requestTokenUrl = $oauth->getAdapter()->getRequestTokenUrl();
header("location: $requestTokenUrl");
```

将consumerKey和consumerSecret替换为在豆瓣申请应用的API Key与Secret，然后访问

```
http://localhost/EvaOAuth/examples/request.php
```

不出意外的话会被引导向豆瓣进行授权。

这一步中，我们取得了一个Request Token，然后将其暂存在Session里。然后被跳转往第三方网站进行授权。

虽然Request Token只存在于OAuth1.0规范，但是为了兼容两个规范，即便是OAuth2.0中，EvaOAuth也会构建一个虚拟的Request Token。

授权后会被带往我们指定的链接callbackUrl。


###用Request Token换取Access Token

继续编写另一个文件access.php

```php
require_once './vendor/autoload.php';
use EvaOAuth\Service as OAuthService;

$oauth = new OAuthService();
$oauth->setOptions(array(
    'callbackUrl' => 'http://localhost/EvaOAuth/examples/access.php',
    'consumerKey' => 'XXX',
    'consumerSecret' => 'YYY',
));
$oauth->initAdapter('Douban', 'OAuth2');

$requestToken = $oauth->getStorage()->getRequestToken();
$accessToken = $oauth->getAdapter()->getAccessToken($_GET, $requestToken);
$accessTokenArray = $oauth->getAdapter()->accessTokenToArray($accessToken);
$oauth->getStorage()->saveAccessToken($accessTokenArray);
$oauth->getStorage()->clearRequestToken();

print_r($accessTokenArray);
```

在这一步中，从Session中取出上一步获得的Request Token，配合CallbackUrl中携带的参数，最终会换取一个授权的Access Token。上例中我们会看到最终获得的Access Token信息：

```
    Array (
    [adapterKey] => douban
    [token] => tokenXXXXXXX
    [expireTime] => 2012-12-06 15:20:38
    [refreshToken] => refreshTokenXXXXXX
    [version] => OAuth2
    [remoteUserId] => 1291360
    )
```

###使用Access Token访问API

取得Access Token后，我们可以根据需求将其存入数据库或以其他方式存放。如果需要携带Access Token访问API也很简单，比如使用上例中的$accessTokenArray：


```php
$oauth = new OAuthService();
$oauth->setOptions(array(
    'consumerKey' => 'XXX',
    'consumerSecret' => 'YYY',
));
$oauth->initByAccessToken($accessTokenArray);
$adapter = $oauth->getAdapter();

$client = $adapter->getHttpClient();
$client->setUri('https://api.douban.com/v2/user/~me');
$response = $client->send();
print_r($response->getBody());
```


Access Token格式参考
-----------

EvaOAuth最终返回的Access Token格式是统一的，但是由于第三方应用规定的差别，并不是所有的参数都一定存在：

- adapterKey (Required) ： 第三方网站名，全小写 
- token (Required)  ： Access Token
- tokenSecret ： Access Token Secret，仅在OAuth1.0中存在
- version (Required)  ： 值为 OAuth1/OAuth2
- refreshToken ： Refresh Token
- expireTime ： Access Token过期时间，为UTC时间
- remoteUserId (Required) ： 当前用户在第三方网站的User Id
- remoteUserName ： 当前用户在第三方网站的User Name
- remoteExtra : 取得Access Token时的其他附加信息，如果有则为一个Json字符串

OAuth登录判断
-----------

每次用户的OAuth登录，只需要判定adapterKey/version/remoteUserId三个值完全一致时，即可认为是同一用户。


注意事项
----------

很多第三方应用内需要将测试用的域名加入白名单。

Yahoo OAuth必须在App Permissions栏选择并设定至少一项权限，否则会出现oauth_problem=consumer_key_rejected错误


由于Zend Http新版本存在Bug，可能会引起

```
    Fatal error: Call to a member function connect() on a non-object in Zend/Http/Client.php 
```

这样的报错，目前修复的方法是强制使用旧版本的Zend Http，项目composer.json中指定：

```
    "allovince/evaoauth": "dev-master",
    "zendframework/zend-http": "2.2.3",
```

重新运行composer即可

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
title: EvaOAuth0.1 文档
---

### EvaOAuth1.0 已经发布，请去[新版本页面](http://avnpc.com/pages/evaoauth)查看内容，本页内容不再维护


[EvaOAuth](http://avnpc.com/pages/evaoauth)是一个统一接口设计，兼容 OAuth1.0 与 OAuth2.0 规范的 php oauth 登录模块，目前支持超过 20 个主流网站的 OAuth 登录，包括：

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
  5. 腾讯 QQ Tencent OAuth2
  6. 开心网 Kaixin OAuth2
  7. 百度 Baidu OAuth2
  8. 360 Qihoo OAuth2
  9. 网易微博 Netease OAuth2
  10. 搜狐微博 Sohu OAuth1
  11. 天涯 Tianya OAuth1

EvaOAuth 统一接口规范，上面的任何一个第三方网站，在使用 EvaOAuth 时的代码与流程都是完全一致的，也可以很简单的扩展并加入新的第三方网站。

最终可以用 20 行左右代码实现以上所有支持网站的完整 OAuth 登录授权。


获得代码
-------------

推荐在项目中使用 Composer 进行一键安装。

请编辑 composer.json，加入

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

EvaOAuth 要求 PHP 版本必须高于 5.3.3，并主要依赖以下几个 ZF2 模块：

- [ZendOAuth](https://github.com/zendframework/ZendOAuth)
- Zend\Session 储存 Token 信息，也可以自己扩展 Storage\StorageInterface 实现其他存储方式
- Zend\Json

在同目录下会自动创建 vendor 目录并下载所有的依赖，在你的项目中，只需要包含自动生成的 vendor/autoload.php 即可。

或者请访问[EvaOAuth 的 Github 项目主页](https://github.com/AlloVince/EvaOAuth)。

###在 Windows 环境下 composer.phar 的安装配置

参考之前的[ZF2 在 Windows 下的环境搭建](http://avnpc.com/pages/zend-framework-2-installation-for-windows)，假设我们的 php.exe 目录在 d:\xampp\php，那么首先将 php 目录加入 windows 环境变量。

```plain
    cd d:\xampp\php
    php -r "eval('?>'.file_get_contents('https://getcomposer.org/installer'));"
```


同目录下编辑文件 composer.bat，内容为

```plain
    @ECHO OFF
    SET composerScript=composer.phar
    php "%~dp0%composerScript%" %*
```

运行

```plain
    composer -V
```
检查 composer 安装是否成功。

进入 EvaOAuth 目录下运行：

```plain
    php D:\xampp\php\composer.phar install
```


申请应用
-------------

实现 OAuth 登录必须先在相应的第三方网站上申请应用并获得的 consumer key 与 consumer secret，每个网站可能叫法不太一样，以豆瓣为例：

访问[豆瓣开发者](http://developers.douban.com/)，我的应用->创建新应用。创建完毕后

- 豆瓣应用的 API Key 对应 EvaOAuth 的 consumer key
- 豆瓣应用的 Secret 对应 EvaOAuth 的 consumer secret

快速开始
-------------

假设我们将 EvaOAuth 文件夹命名为 OAuth 并可以用http://localhost/OAuth/访问，同时已经安装好了所有的依赖。我们以豆瓣的 OAuth2.0 为例（因为豆瓣没有限制 CallbackUrl 的域，非常方便测试），用几十行代码构建一个完整的 OAuth 登录：

###获得 Request Token 并跳转到第三方进行授权

首先编写一个文件 request.php，内容如下：

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

将 consumerKey 和 consumerSecret 替换为在豆瓣申请应用的 API Key 与 Secret，然后访问

```plain
http://localhost/EvaOAuth/examples/request.php
```

不出意外的话会被引导向豆瓣进行授权。

这一步中，我们取得了一个 Request Token，然后将其暂存在 Session 里。然后被跳转往第三方网站进行授权。

虽然 Request Token 只存在于 OAuth1.0 规范，但是为了兼容两个规范，即便是 OAuth2.0 中，EvaOAuth 也会构建一个虚拟的 Request Token。

授权后会被带往我们指定的链接 callbackUrl。


###用 Request Token 换取 Access Token

继续编写另一个文件 access.php

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

在这一步中，从 Session 中取出上一步获得的 Request Token，配合 CallbackUrl 中携带的参数，最终会换取一个授权的 Access Token。上例中我们会看到最终获得的 Access Token 信息：

```plain
    Array (
    [adapterKey] => douban
    [token] => tokenXXXXXXX
    [expireTime] => 2012-12-06 15:20:38
    [refreshToken] => refreshTokenXXXXXX
    [version] => OAuth2
    [remoteUserId] => 1291360
    )
```

###使用 Access Token 访问 API

取得 Access Token 后，我们可以根据需求将其存入数据库或以其他方式存放。如果需要携带 Access Token 访问 API 也很简单，比如使用上例中的$accessTokenArray：


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


Access Token 格式参考
-----------

EvaOAuth 最终返回的 Access Token 格式是统一的，但是由于第三方应用规定的差别，并不是所有的参数都一定存在：

- adapterKey (Required) ： 第三方网站名，全小写 
- token (Required)  ： Access Token
- tokenSecret ： Access Token Secret，仅在 OAuth1.0 中存在
- version (Required)  ： 值为 OAuth1/OAuth2
- refreshToken ： Refresh Token
- expireTime ： Access Token 过期时间，为 UTC 时间
- remoteUserId (Required) ： 当前用户在第三方网站的 User Id
- remoteUserName ： 当前用户在第三方网站的 User Name
- remoteExtra : 取得 Access Token 时的其他附加信息，如果有则为一个 Json 字符串

OAuth 登录判断
-----------

每次用户的 OAuth 登录，只需要判定 adapterKey/version/remoteUserId 三个值完全一致时，即可认为是同一用户。


注意事项
----------

很多第三方应用内需要将测试用的域名加入白名单。

Yahoo OAuth 必须在 App Permissions 栏选择并设定至少一项权限，否则会出现 oauth_problem=consumer_key_rejected 错误


由于 Zend Http 新版本存在 Bug，可能会引起

```plain
    Fatal error: Call to a member function connect() on a non-object in Zend/Http/Client.php 
```

这样的报错，目前修复的方法是强制使用旧版本的 Zend Http，项目 composer.json 中指定：

```plain
    "allovince/evaoauth": "dev-master",
    "zendframework/zend-http": "2.2.3",
```

重新运行 composer 即可

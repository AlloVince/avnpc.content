---
published: true
date: '2015-05-01 17:21:07'
tags:
  - php
  - OAuth
author: AlloVince
title: EvaOAuth 1.0：统一接口的 OAuth 登录 PHP 类库
---

[![Latest Stable Version](https://poser.pugx.org/evaengine/eva-oauth/v/stable.svg)](https://packagist.org/packages/evaengine/eva-oauth)
[![License](https://poser.pugx.org/evaengine/eva-oauth/license.svg)](https://packagist.org/packages/evaengine/eva-oauth)
[![Build Status](https://travis-ci.org/AlloVince/EvaOAuth.svg?branch=feature%2Frefactoring)](https://travis-ci.org/AlloVince/EvaOAuth)
[![Coverage Status](https://coveralls.io/repos/AlloVince/EvaOAuth/badge.svg?branch=master)](https://coveralls.io/r/AlloVince/EvaOAuth?branch=master)
[![Scrutinizer Code Quality](https://scrutinizer-ci.com/g/AlloVince/EvaOAuth/badges/quality-score.png?b=master)](https://scrutinizer-ci.com/g/AlloVince/EvaOAuth/?branch=master)

EvaOAuth 是一个统一接口设计的[PHP OAuth](http://avnpc.com/pages/evaoauth) Client 库，兼容 OAuth1.0 与 OAuth2.0 规范，可以通过 10 多行代码集成到任意项目中。

项目代码托管在 [https://github.com/AlloVince/EvaOAuth](https://github.com/AlloVince/EvaOAuth) ，欢迎 Star 及 Fork 贡献代码。

## 为什么选择 EvaOAuth 

经过若干项目考验， EvaOAuth1.0 根据实际需求进行了一次完全重构，主要的一些特性如下：

- **标准接口**，无论 OAuth1.0 或 OAuth2.0，同一套代码实现不同工作流，并且获取一致的数据格式，包括用户信息和 Token。  
- **充分测试**，所有关键代码进行单元测试，同时通过 CI 保证多版本 PHP 下的可用性。 
- **容易调试**，开启 Debug 模式后，Log 中会记录 OAuth 流程中所有的 URL、Request、Response，帮助定位问题。
- **开箱即用**，项目已经内置了主流的 OAuth 网址支持，如微博、QQ、Twitter、Facebook 等。
- **方便扩展**，可以通过最少 3 行代码集成新的 OAuth 服务，工作流程提供事件机制。 

## 快速开始

EvaOAuth 可以通过[Packagist](https://packagist.org/packages/evaengine/eva-oauth)下载，推荐通过 Composer 安装。

编辑 composer.json 文件为：

``` json
{
    "require": {
        "evaengine/eva-oauth": "~1.0"
    }
}
```

然后通过 Composer 进行安装。

``` shell
curl -sS https://getcomposer.org/installer | php
php composer.phar install
```

下面通过一个实例演示如何集成豆瓣登录功能。假设已经在[豆瓣开发者](http://developers.douban.com)创建好一个应用。准备一个 request.php 如下：

``` php
require_once './vendor/autoload.php'; //加载Composer自动生成的autoload
$service = new Eva\EvaOAuth\Service('Douban', [
    'key' => 'You Douban App ID',  //对应豆瓣应用的API Key
    'secret' => 'You Douban App Secret', //对应豆瓣应用的Secret
    'callback' => 'http://localhost/EvaOAuth/example/access.php' //回调地址
]);
$service->requestAuthorize();
```

在浏览器中运行 request.php，如果参数正确则会被重定向到豆瓣授权页面，登录授权后会再次重定向回我们设置的`callback`。因此再准备好 access.php 文件：

``` php
$token = $service->getAccessToken();
```

这样就拿到了豆瓣的 Access Token，接下来可以使用 Token 去访问受保护的资源：

``` php
$httpClient = new Eva\EvaOAuth\AuthorizedHttpClient($token);
$response = $httpClient->get('https://api.douban.com/v2/user/~me');
```
 
这样就完成了 OAuth 的登录功能。更多细节可以参考代码的[示例](https://github.com/AlloVince/EvaOAuth/tree/master/examples)以及[Wiki](https://github.com/AlloVince/EvaOAuth/wiki)页面。

## OAuth 网站支持

EvaOAuth 将一个 OAuth 网站称为一个 Provider。目前支持的 Provider 有：

- OAuth2.0
  - 豆瓣（Douban）
  - Facebook
  - QQ （Tencent）
  - 微博 （Weibo）
- OAuth1.0
  - Twitter
  
新增一个 Provider 仅需数行代码，下面演示如何集成 Foursquare 网站：


``` php
namespace YourNamespace;

class Foursquare extends \Eva\EvaOAuth\OAuth2\Providers\AbstractProvider
{
    protected $authorizeUrl = 'https://foursquare.com/oauth2/authorize';
    protected $accessTokenUrl = 'https://foursquare.com/oauth2/access_token';
}
```

然后将 Provider 注册到 EvaOAuth 就可以使用了。

``` php
use Eva\EvaOAuth\Service;
Service::registerProvider('foursquare', 'YourNamespace\Foursquare');
$service = new Service('foursquare', [
    'key' => 'Foursquare App ID',
    'secret' => 'Foursquare App Secret',
    'callback' => 'http://somecallback/'
]);
```

## 数据存储

在 OAuth1.0 的流程中，需要将 Request Token 保存起来，然后在授权成功后使用 Request Token 换取 Access Token。因此需要数据存储功能。

EvaOAuth 的数据存储通过[Doctrine\Cache](https://github.com/doctrine/cache)实现。默认情况下 EvaOAuth 会将数据保存为本地文件，保存路径为`EvaOAuth/tmp`。

可以在 EvaOAuth 初始化前任意更改存储方式及存储位置，例如将文件保存位置更改为`/tmp`：

``` php
Service::setStorage(new Doctrine\Common\Cache\FilesystemCache('/tmp'));
```

或者使用 Memcache 保存：

``` php
$storage = new \Doctrine\Common\Cache\MemcacheCache();
$storage->setMemcache(new \Memcache());
Service::setStorage($storage);
```

## 事件支持

EvaOAuth 定义了若干事件方面更容易的注入逻辑

- `BeforeGetRequestToken`: 获取 Request Token 前触发。
- `BeforeAuthorize`: 重定向到授权页面前触发。
- `BeforeGetAccessToken`: 获取 Access Token 前触发。

比如我们希望在获取 Access Token 前向 HTTP 请求中加一个自定义 Header，可以通过以下方式实现：

``` php
$service->getEmitter()->on('beforeGetAccessToken', function(\Eva\EvaOAuth\Events\BeforeGetAccessToken $event) {
    $event->getRequest()->addHeader('foo', 'bar');
});
```

## 技术实现

EvaOAuth 基于强大的 HTTP 客户端库[Guzzle](https://github.com/guzzle/guzzle)，并通过 OOP 方式对 OAuth 规范进行了完整的描述。

为了避免对规范的诠释上出现误差，底层代码优先选择规范描述中的角色与名词，规范间差异则在上层代码中统一。

因此如果没有同时支持两套规范的需求，可以直接使用 OAuth1.0、OAuth2.0 分别对应的工作流。

详细用例可以参考 Wiki：
 
- [OAuth1.0](https://github.com/AlloVince/EvaOAuth/wiki/OAuth1.0-Specification-Implementation)
- [OAuth2.0](https://github.com/AlloVince/EvaOAuth/wiki/OAuth2.0-Specification-Implementation)

## Debug 与 Log

开启 Debug 模式将在 Log 中记录所有的请求与响应。

``` php
$service->debug('/tmp/access.log');
```

请确保 PHP 对 log 文件有写入权限。

## API 文档

首先通过`pear install phpdoc/phpDocumentor`安装 phpDocumentor，然后在项目根目录下运行`phpdoc`，会在`docs/`下生成 API 文档。

## 问题反馈及贡献代码

项目代码托管在 [https://github.com/AlloVince/EvaOAuth](https://github.com/AlloVince/EvaOAuth) ，欢迎 Star 及 Fork 贡献代码。

有问题欢迎在[EvaOAuth Issue](https://github.com/AlloVince/EvaOAuth/issues)提出。




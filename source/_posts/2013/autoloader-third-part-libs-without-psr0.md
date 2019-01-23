---
published: true
date: '2013-01-11 15:56:07'
tags:
  - ZF2
  - Zend Framework 2
  - GeoIP
  - PSR-0
author: AlloVince
title: ZF2 自动加载非 PSR-0 标准库及实例（GeoIP 地理位置查询）
---

Zend Framework 2.0 的自动加载机制主要基于[PSR-0 标准](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-0.md)，引入新的第三方库只需要注册新的命名空间即可。

比如加载[ZendOAuth 模块](https://github.com/zendframework/ZendOAuth)，下载后放于 vendor/ZendOAuth，然后在依赖的模块 Module.php 中加入一行即可：

``` php
public function getAutoloaderConfig()
{
    return array(
        'Zend\Loader\StandardAutoloader' => array(
            'namespaces' => array(
                'ZendOAuth' => __DIR__ . '/../../vendor/ZendOAuth/library/ZendOAuth',
                __NAMESPACE__ => __DIR__ . '/src/' . __NAMESPACE__,
            ),
        ),
    );
}
```


## ZF2 加载非 PSR-0 标准库

不过由于第三方库质量参差不齐，并不一定所有的库都遵循 PSR-0 标准，[ZF2 也提供了非 PSR-0 标准库的自动加载](http://avnpc.com/pages/autoloader-third-part-libs-without-psr0)方式，可以在 Zend\Loader 中新增[ClassMapAutoloader](http://framework.zend.com/manual/2.0/en/modules/zend.loader.class-map-autoloader.html)，以键值对的形式引入需要加载的第三方库文件。下面以一个正好碰到的实例说明：


### GeoIP 地理位置查询+ZF2 实例

项目中正好有一个简单的需求，要根据用户 IP 显示用户所在地，IP 来源可能是全球范围，所以需要找一个 IP 与地理位置对应的数据库：[GeoIP](http://avnpc.com/pages/autoloader-third-part-libs-without-psr0)

GeoIP 数据库由[MaxMind 公司](http://www.maxmind.com/en/home)负责维护，免费版可以获得精确到城市一级的 IP 与地理位置对应数据，对于一般的应用来说已经够用了。

首先需要下载[GeoIP 数据库](http://dev.maxmind.com/geoip/geolite)，选择[Binary GeoLite City 版](http://geolite.maxmind.com/download/geoip/database/GeoLiteCity.dat.gz)，同时还需要下载[PHP GeoIP 数据库接口](http://dev.maxmind.com/geoip/downloadable#PHP-7)，为了更好的兼容性选择了[纯 php 版本的 GeoIP 接口](http://www.maxmind.com/download/geoip/api/php/)

最后项目中这样放置：

```plain
project/
--data/
----GeoLiteCity.dat
--module/
----MyModule/
------Module.php
------vendor/
--------GeoIP/
----------geoip.inc
----------geoipcity.inc
----------geoipregionvars.php
```

在模块 Module.php 中首先引入 ClassMapAutoloader，并在同目录下放置一个 autoload_classmap.php 文件

``` php
public function getAutoloaderConfig()
{
    return array(
        'Zend\Loader\ClassMapAutoloader' => array(
            __DIR__ . '/autoload_classmap.php',
        ),
        'Zend\Loader\StandardAutoloader' => array(
            'namespaces' => array(
                __NAMESPACE__ => __DIR__ . '/src/' . __NAMESPACE__,
            ),
        ),
    );
}
```

在 autoload_classmap.php 文件中需要制定第三方库文件的位置

``` php
<?php
return array(
    'GeoIP' => __DIR__ . '/vendor/GeoIP/geoipcity.inc',
);
```

最后在模块 Controller 中就可以通过自动加载机制直接使用：

``` php
public function geoAction()
{
    $city = 'unkown';
    $geoData = __DIR__ . '/../../../../../data/databases/GeoLiteCity.dat';
    new \GeoIP();
    $ip = $_SERVER['REMOTE_ADDR'];
    $gi = geoip_open($geoData, GEOIP_STANDARD);
    $record = geoip_record_by_addr($gi, $ip);
    if(isset($record->city)){
        $city = $record->city;
    }
    geoip_close($gi);
    return array(
        'city' => $city
    );
}
```


## ZF2 自动加载下划线分割的 PSR-0 类库

以下划线作为类名分隔符也很常见，这一种类库的加载在 ZF2 中仍然使用 StandardAutoloader，不过需要指定的不是 Namespaces 而是 Prefixes，比如加载[PHP-Resque](https://github.com/chrisboulton/php-resque)这个项目，在 Module.php 中加入：

``` php
public function getAutoloaderConfig()
{
  return array(
	 'Zend\Loader\ClassMapAutoloader' => array(
		__DIR__ . '/autoload_classmap.php',
	 ),
	 'Zend\Loader\StandardAutoloader' => array(
		'prefixes' => array(
		   'Resque' => __DIR__ . '/../../vendor/Resque/lib/Resque',
		),
	 ),
  );
}
```
   
同时在 autoload_classmap 中加入

``` php
return array(
    'Resque' => __DIR__ . '/../../vendor/Resque/lib/Resque.php',
);
```

即可。



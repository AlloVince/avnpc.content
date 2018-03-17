---
slug: autoloader-third-part-libs-without-psr0
date: '2013-01-11 15:56:07'
title: ZF2自动加载非PSR-0标准库及实例（GeoIP地理位置查询）
id: 172
tags:
  - ZF2
  - Zend Framework 2
  - GeoIP
  - PSR-0
---

Zend Framework 2.0的自动加载机制主要基于[PSR-0标准](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-0.md)，引入新的第三方库只需要注册新的命名空间即可。

比如加载[ZendOAuth模块](https://github.com/zendframework/ZendOAuth)，下载后放于vendor/ZendOAuth，然后在依赖的模块Module.php中加入一行即可：

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


## ZF2加载非PSR-0标准库

不过由于第三方库质量参差不齐，并不一定所有的库都遵循PSR-0标准，[ZF2也提供了非PSR-0标准库的自动加载](http://avnpc.com/pages/autoloader-third-part-libs-without-psr0)方式，可以在Zend\Loader中新增[ClassMapAutoloader](http://framework.zend.com/manual/2.0/en/modules/zend.loader.class-map-autoloader.html)，以键值对的形式引入需要加载的第三方库文件。下面以一个正好碰到的实例说明：


### GeoIP地理位置查询+ZF2实例

项目中正好有一个简单的需求，要根据用户IP显示用户所在地，IP来源可能是全球范围，所以需要找一个IP与地理位置对应的数据库：[GeoIP](http://avnpc.com/pages/autoloader-third-part-libs-without-psr0)

GeoIP数据库由[MaxMind公司](http://www.maxmind.com/en/home)负责维护，免费版可以获得精确到城市一级的IP与地理位置对应数据，对于一般的应用来说已经够用了。

首先需要下载[GeoIP数据库](http://dev.maxmind.com/geoip/geolite)，选择[Binary GeoLite City版](http://geolite.maxmind.com/download/geoip/database/GeoLiteCity.dat.gz)，同时还需要下载[PHP GeoIP数据库接口](http://dev.maxmind.com/geoip/downloadable#PHP-7)，为了更好的兼容性选择了[纯php版本的GeoIP接口](http://www.maxmind.com/download/geoip/api/php/)

最后项目中这样放置：

```
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

在模块Module.php中首先引入ClassMapAutoloader，并在同目录下放置一个autoload_classmap.php文件

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

在autoload_classmap.php文件中需要制定第三方库文件的位置

``` php
<?php
return array(
    'GeoIP' => __DIR__ . '/vendor/GeoIP/geoipcity.inc',
);
```

最后在模块Controller中就可以通过自动加载机制直接使用：

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


## ZF2自动加载下划线分割的PSR-0类库

以下划线作为类名分隔符也很常见，这一种类库的加载在ZF2中仍然使用StandardAutoloader，不过需要指定的不是Namespaces而是Prefixes，比如加载[PHP-Resque](https://github.com/chrisboulton/php-resque)这个项目，在Module.php中加入：

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
   
同时在autoload_classmap中加入

``` php
return array(
    'Resque' => __DIR__ . '/../../vendor/Resque/lib/Resque.php',
);
```

即可。


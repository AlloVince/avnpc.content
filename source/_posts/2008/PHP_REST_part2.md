---
title: "基于Picasa、PHP、AJAX的图片REST架构的简单实现(2)"
date: "2008-06-27 15:39:08"
slug: "PHP_REST_part2"
published: true
author: "AlloVince"
legacy_id: 99
comment_status: "closed"
comments: false
tags:
  - "Curl"
  - "Mod_Rewrite"
  - "PEAR"
  - "php"
  - "Picasa"
  - "REST"
  - "XML_Serializer"
  - "YD的程序员葛阁"
---
前情提要：[基于Picasa、PHP、AJAX的图片REST架构的简单实现](blog/view/98)

在前面展示的[DEMO](lab/photo_rest/demo3.html)中，迫不及待的同学或许已经打开图片链接看过了，但很遗憾，这张图片并不存在于ave7.net的服务器上。我们首先要了解的是标题中的第一个主角Picasa。

来自Google的Picasa无疑是目前互联网最富有代表性的网络相册之一，而我选择Picasa而不是其他风头更劲的Flickr等等的原因有三：

1. Google的服务很稳定
2. API有很大发挥的余地
3. 强劲的客户端支持

Picasa的API文档在[这里](http://code.google.com/apis/picasaweb/overview.html)，有兴趣的同学可以自行慢慢研究，关于本文所涉及的内容，我们只需要知道两点：

- Picasa的API是以RSS形式发布的
- 在Picasa中准确定位一张图片需要三个参数:用户名、相册id、相片id

所以在上面的URL中，我们引入这三个参数，经由解析Picasa的API，就可以得到这张图片的所有信息：缩略图、原图、拍摄地点、标签……不一而足。而上述URL，则是我们提到的服务器端的请求分析器，用PHP来实现。

#### Step.1 服务器端解析Picasa API并显示缩略图

为了更好完成这个目标，这里涉及的知识有

1. 借助Apache的[URL Mod_Rewrite](http://httpd.apache.org/docs/2.2/rewrite/)模块来规整URL；
2. 利用PHP的[Curl](http://php.net/manual/en/book.curl.php)函数组来获取远端文件；
3. 用PHP的[PEAR](http://pear.php.net/)库中的[XML_Serializer](http://pear.php.net/package/XML_Serializer)来解析XML文件；

当然本文的重点不是详细讲解以上这些技术，编码上的细节问题会一笔带过，文章的核心始终是框架的搭建。

假设服务器PHP文件名为_loadpic.php，在mod_rewrite模块开启后，同目录下建立一个.htaccess文件，内容为

```php
RewriteEngine On

RewriteRule   ^photo\/picasa\/(\w+)\/(\d+)\/(\d+)    _loadpic.php?userid=$1&albumid=$2&photoid=$3
```

\_loadpic.php文件的内容为

```php
//引入XML_Serializer
require_once 'XML/Unserializer.php';
$Unserializer = &new XML_Unserializer();
$Unserializer->setOptions(array('parseAttributes'   => true));

if($userid && $albumid && $photoid) {
    $xml = "http://picasaweb.google.com/data/feed/api/user/".$_GET['userid']."/albumid/".$_GET['albumid']."/photoid/".$_GET['photoid'];
    $status = $Unserializer->unserialize($xml,true);
    //XML文件解析出错, 报错并退出
    if (PEAR::isError($status)) {
       die($status->getMessage());
       exit;
    }
    $pic = $Unserializer->getUnserializedData();
    header("Content-Type: image/jpeg");
    //Curl获得远端文件，Curl模块需要在PHP.ini的单独开启。
    curl_exec(curl_init($pic['icon']));
    }
}
```

而通过以上处理，我们达到了初步的目标：通过用户名、相册id、相片id三个参数定位一张照片，访问并解析Picasa API并获得我们需要的图片，然后显示出来。

[这里](lab/photo_rest/demo1.html)是我们初步实现的结果。

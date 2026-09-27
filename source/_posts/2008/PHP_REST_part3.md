---
title: "基于Picasa、PHP、AJAX的图片REST架构的简单实现(3)"
date: "2008-07-23 16:11:44"
slug: "PHP_REST_part3"
published: true
author: "AlloVince"
legacy_id: 104
comment_status: "closed"
comments: false
tags:
  - "HTTP Headers"
  - "php"
  - "REST"
  - "YD的程序员葛阁"
---
前情提要：[基于Picasa、PHP、AJAX的图片REST架构的简单实现](blog/view/98)

之前我们从远端获得了图片信息，但是这些和REST还是不沾边的。访问同一个URL，要想获得不同的信息，以目前的软硬件条件，最好的实现方法还是对HTTP Headers进行处理。所以这次的话题是——

#### Step.2 服务器端接受不同的HTTP Headers返回不同的信息

既然是服务器端，我们仍然采用PHP实现。

PHP对于Header信息的处理是很丰富的，常见的Header都已经存放在预定义变量$\_SERVER中了。如果你对什么是Header还没有概念，可以在Firefox中安装一个[Live HTTP Headers](https://addons.mozilla.org/en-US/firefox/addon/3829)自行体验一下。

现在问题是：如此之多的Header，到底选择哪个或者哪几个来进行分析？用户的客户端语言，浏览器，IP等等，都是可以进行发挥的。这里为了简单说明问题，选择REQUEST_METHOD，HTTP_REFERER以及HTTP_X_REQUESTED_WITH来进行演示。

简单说明这三个参数：

- REQUEST_METHOD是页面的请求方法，也是目前REST实现的核心。
- HTTP_REFERER包含了用户的访问来源，一般是用户前一个访问的URL
- HTTP_X_REQUESTED_WITH则代表这是一个Xmlhttprequest请求，可以简单的看成是Ajax请求。

以下就是个人的随意发挥了，我所约定的有效服务器反馈包括：

- 因为不涉及更新等操作，页面的请求方法只限于GET
- 如果HTTP_REFERER存在，即说明用户在网页中或者通过链接点击访问照片，服务器返回照片的缩略图。
- 如果HTTP_REFERER不存在，说明用户直接通过URL访问图片，服务器返回照片的全图。
- 如果HTTP_X_REQUESTED_WITH存在，则说明用户在通过Ajax访问图片，这时返回一个Json格式的照片信息

形象一点用实例说明的话：

[点击链接](http://allo.ave7.net/lab/photo_rest/photo/picasa/allovince/5213571636180988625/5213571644436936978)会看到照片的缩略图，将这张照片的地址粘贴到浏览器地址栏直接访问的话，会看到照片的大图。而在[Demo](lab/photo_rest/demo2.html)中，点击下面的按钮，则可以看到发送Ajax请求到同一个地址时的反馈。

最后还是附上服务器端的代码

```php
require_once 'XML/Unserializer.php';

$Unserializer = &new XML_Unserializer();
$options = array('parseAttributes'   => true);
$Unserializer->setOptions($options);

if($userid && $albumid && $photoid) {
	$xml = "http://picasaweb.google.com/data/feed/api/user/".$_GET['userid']."/albumid/".$_GET['albumid']."/photoid/".$_GET['photoid'];
	$status = $Unserializer->unserialize($xml,true);
	//XML文件解析出错, 报错并退出
	if (PEAR::isError($status)) {
	   die($status->getMessage());
	   exit;
	}
	//如果是一个AJAX访问，返回JSON描述
	elseif($_SERVER['REQUEST_METHOD'] == 'GET' && $_SERVER['HTTP_X_REQUESTED_WITH']){
		$pic = $Unserializer->getUnserializedData();
		$part = explode('.',$pic['media:group']['media:content']['url']);
		$json = json_encode(array(
			'title' => $pic['media:group']['media:description']['_content'] ? $pic['media:group']['media:description']['_content'] : $pic['title']['_content'],
			'date' => gmdate("F d Y H:i:s",strtotime($pic['updated'])),
			'ext' => $part[count($part) -1],
			'full' => $pic['media:group']['media:content']['url']
		));
		echo "($json)";
	}
	//HTTP_REFERER有无的区分
	elseif($_SERVER['REQUEST_METHOD'] == 'GET' && $_SERVER['HTTP_REFERER']) {
		$pic = $Unserializer->getUnserializedData();
		header("Content-Type: image/jpeg");
		curl_exec(curl_init($pic['icon']));
	}
	elseif($_SERVER['REQUEST_METHOD'] == 'GET' && !$_SERVER['HTTP_REFERER']) {
		$pic = $Unserializer->getUnserializedData();
		header("Content-Type: ".$pic['media:group']['media:content']['type']);
		curl_exec(curl_init($pic['media:group']['media:content']['url']));
	}
}
```

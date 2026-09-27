---
title: "基于Picasa、PHP、AJAX的图片REST架构的简单实现 完结篇"
date: "2008-07-24 15:49:47"
slug: "PHP_REST_part4"
published: true
author: "AlloVince"
legacy_id: 105
comment_status: "closed"
comments: false
tags:
  - "Ajax"
  - "javascript"
  - "jQuery"
  - "XMLHttpRequest"
  - "YD的程序员葛阁"
  - "翻墙"
---
一如既往的前情提要：[基于Picasa、PHP、AJAX的图片REST架构的简单实现](blog/view/98)

现在，服务器端的架设终于全部结束了，我们可以向服务器端同一个URL发送不同的Header请求，从而得到不同的反馈，REST简单说来也就是这么一回事。但很遗憾，URL只有一个，而且现在的浏览器并没有智能到可以通过超链接携带Header信息。我们只能通过附加手段达到目的，这个附加手段毫无疑问就是最后一位主角：Ajax。

#### Step.3 客户端通过Ajax发送不同的Header请求

准确的说我们利用的是[XMLHttpRequest](http://www.w3.org/TR/XMLHttpRequest/)对象，来对服务器发送不同的header请求。在W3C的定义中，不仅能够找到GET、POST，XMLHttpRequest支持的header类型一共包括

- GET
- POST
- HEAD
- PUT
- DELETE
- OPTIONS

这些Methods早在1999年就被定义于[RFC2616](http://www.ietf.org/rfc/rfc2616)里，到今天终于有了用武之地，真是可喜可贺。

```js
var client = new XMLHttpRequest();
client.open("GET", "test.txt", true);
client.send();
```

这就是一个最简单的通过XMLHttpRequest发送请求的例子，同理可以将GET替换为其他Method。如果再加上监听状态变化的readystatechange，以及获得反馈的responseXXX，你会发现Ajax原来就是这么一回事。

这里为了使演示更加简单，使用JQuery的Ajax应用，以下是客户端代码:

```js
	$("img").each(function(){
		var src = $(this).attr("src");
		var aim = $(this);
		$.ajax({
			type: "GET",
			url: src,
			dataType : 'json',
			success: function(json){
				var title= "<p>这是一张名为「" + json.title + "」的相片<br />拍摄于：" + json.date + "<br />你可以访问：" + json.full + "来获得这张照片的全图。</p>";
				aim.after(title);
				aim.attr("alt",json.title);
			}
		});
		return false;
	});
```

这段代码遍历页面的所有图片，然后对这张图片的URL发送Get请求。发送的请求被我们编写的服务器端接收并取得对应的Json数据并返回，最终在客户端显示出来。

至此，这个最简单的图片REST架构便完成了，或许花这么大功夫最终只显示出一张图片来，还称之为构架，实在是小题大作。但最终能剔除所有冗余信息并实现HTML代码的最简化，我自己是玩的不亦乐乎的。

最后附上[全部代码下载](http://cid-01e48df64f8bd957.skydrive.live.com/embedrowdetail.aspx/Source/photorest.7z)，也可以[在线查看](lab/photo_rest/)。

##### 付录：HTTP-Header最高绝技之翻墙术

其实在国内有很多AV国的网站上不了，并不是GFW的功劳，而网站为了避免拦截正常的访问，也很少会去封IP段。那么究竟为什么很多XX站不能访问呢，在HTTP-Header中，有一个叫Accept-Language的字段负责描述用户的系统语言，国内的Accept-Language值一般都是zh-cn,zh，AV国的很多网站都是通过识别这个值来区别用户。

那么很简单，更改Accept-Language伪装成AV国用户就可以了。

- Firefox下，地址栏访问about:config，然后搜索intl.accept_languages，将值由zh-cn,zh改为ja即可。
- IE下，工具——Internet选项——语言——将列表里的语言全部删除即可。

恩恩，这也只是一个技术问题，大家回去自行研究吧。

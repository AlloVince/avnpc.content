---
title: "基于Picasa、PHP、AJAX的图片REST架构的简单实现 番外篇"
date: "2008-07-30 12:07:45"
slug: "PHP_REST_part5"
published: true
author: "AlloVince"
legacy_id: 106
comment_status: "closed"
comments: false
tags:
  - "javascript"
  - "jQuery"
  - "JSONP"
  - "Picasa"
  - "YD的程序员葛阁"
---
好吧，其实为什么完结了还能再出续集这个问题无需太过深究，就像PB里面的Sara，可以死了再活，活了再死，全凭导演一句话口牙。

回到正题，之前的简单REST实现只能算是一个模型，很难实用到真正的开发中去，瓶颈在于模型中所有远端服务器的数据都先读入到Web服务器，通过PHP脚本处理后再返回客户端。这是一个常规的也是通用的解决方案，因为远端服务器有可能做各种各样的功能限制：检测用户身份，限制访问来源等等，而这些限制都可以通过强大的Curl进行破解。

但终究考虑到服务器资源的占用，这样的实现是不切实际的，既然Picasa已经提供了足够丰富的接口，那么用纯Javascript也能实现近似的效果。当然这样一来作为完全客户端的实现，也就和REST没有任何关系了，所以作为番外篇，仅供参考。

说了这么多，看看完全客户端的实现能做出什么东西来：[点击查看DEMO](lab/photo_restjs/index.html)

Demo中首先显示图片未加载的loading，当图片数据从Picasa加载之后，Loading画面被替换为实际图像的缩略图，并且点击会有Lightbox的特效。

首先通过Url_Rewrite将所有的请求转发到loading图片

```js
RewriteEngine On
RewriteRule   ^photo\/picasa\/.* loading.gif
```

而完成所有页面效果的，只需要14行代码：

```js
$("img").each(function(){
	var src = $(this).attr("src").split("/");
	var aim = $(this);
	$.ajax({
		url:  "http://picasaweb.google.com/data/entry/api/user/" + src[0] + '/albumid/' + src[1] + '/photoid/' + src[2] + '?alt=json',
		dataType : 'jsonp',
		success: function(json){
			var id = 'photo' + src[1] + src[2];
			var title = json.entry.summary.$t ? json.entry.summary.$t : json.entry.title.$t;
			aim.attr("alt",title).attr("src",json.entry.media$group.media$thumbnail[1].url).wrap('<a class="thickbox" id="' + id + '" href="'+ json.entry.media$group.media$content[0].url + '?imgmax=800' + '" rel="album_' + src[1] + '" title="' + title + '"></a>');
			tb_init('#'+ id); //apply thickbox effact to this photo
		}
	});
});
```

大致的实现与[完结篇](blog/view/105)里的代码区别不大，但这里却是完全依赖于Picasa强大API的：

- Picasa支持Jsonp！通过向API设置alt=json可以取得Json格式数据，尽管没有在文档中找到关于可以返回Json数据的描述，但这确确实实是可行的。
- Picasa支持外链！准确的说Picasa对于缩略图的外链是没有限制的，而这里所谓的缩略图，事实上能支持最高达800像素规格的图片（参看[Picasa Web Albums query parameters reference](http://code.google.com/apis/picasaweb/reference.html#Parameters)），对于网页图片浏览来说，已经足够了。

基于以上两点，可以简单通过Js，就能搭建起各种Picasa的扩展应用，同理也可以适用到Flickr以及其他提供Jsonp接口的网络相册。不过这个基于Js的方案也有很大缺点，如果用户禁用了Javascript，就什么也看不到了。

最后仍然是本次实现的[Code Download](http://cid-01e48df64f8bd957.skydrive.live.com/embedrowdetail.aspx/Source/photorestjs.7z)

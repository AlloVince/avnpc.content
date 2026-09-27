---
title: "JQuery TagClouds Plugin"
date: "2008-04-22 17:02:03"
slug: "JQuery_TagClouds_Plugin"
published: true
author: "AlloVince"
legacy_id: 90
comment_status: "closed"
comments: false
tags:
  - "javascript"
  - "jQuery"
  - "TagClouds"
  - "YD的程序员葛阁"
---
**目录** [折叠]

1. [功能](#toc_18833600)
2. [Demo](#toc_18838200)
3. [Sample](#toc_18844600)
4. [Plugin Source](#toc_18850900)

JQuery TagClouds Plugin Ver0.1

#### 功能

将一个元素内的所有链接转换为Tag Clouds（标签云）形式，字体大小和颜色同时渐变。

#### Demo

首页

#### Sample

```js
$(".tag_clouds").tagClouds();
```

更复杂一点的例子：

```js
var option = {
	fontmax:22, //可选 默认值为22, TagCloud的最大fontsize(单位px)
	fontmax:11, //可选 默认值为11, TagCloud的最小fontsize(单位px)
	colorfrom:"#CACAFF", //可选 默认值为"#CACAFF", TagCloud的渐变颜色1(要突出显示的颜色)
	colorto:"#F8B3D0", //可选 默认值为"#F8B3D0", TagCloud的渐变颜色2
	patrn:/\(\d+\)$/ //可选 将Tag权值从链接中分离的正则表达式,默认形式为 标签(10)
}
$(".tag_clouds").tagClouds(option);
```

#### Plugin Source

```js
/*
 * jQuery TagClouds Plugin
 * version: 0.1
 * @requires jQuery v1.2.2 or later
 *
 * Copyright (c) 2008 AlloVince
 * Examples at: http://allo.ave7.net/JQuery_TagClouds_Plugin
 * Licensed under the MIT License:
 *   http://www.opensource.org/licenses/mit-license.php
 *
 */
if(jQuery) (function($){

$.fn.tagClouds = function(option) {
	var fontmax = 21;
	var fontmin = 11;
	var colorfrom = "#CACAFF";
	var colorto = "#F8B3D0";
	var patrn=/\(\d+\)$/;
	if(option) {
		fontmax = option.fontmax ? option.fontmax : fontmax;
		fontmin = option.fontmin ? option.fontmin : fontmin;
		colorfrom = option.colorfrom ? option.colorfrom : colorfrom;
		colorto = option.colorto ? option.colorto : colorto;
		patrn = option.patrn ? option.patrn : colorto;
	}

	var tags = Array();
	var fontsize = Array(),fontcolor = Array();
	var i = 0;
	$(' > a',this).each(function(){
		var tag_count = patrn.exec($(this).text());
		var num=/\d+/;
		tags[i] = num.exec(tag_count);
		i++;
	});
	//数组复制
	var tmp = tags.slice(0);
	tmp.sort(function(a, b) {return b - a;});
	var max = tmp[0];
	var min = tmp[tmp.length-1];
	for(i = 0; i < tags.length ; i++) {
		if(tags[i] == max) {
			fontsize[i] = fontmax;
		}
		else if(tags[i] == min) {
			fontsize[i] = fontmin;
		}
		else {
			fontsize[i] = parseInt( (fontmax - fontmin)*(tags[i] - min)/(max - min) + fontmin);
		}
		fontcolor[i] = color(colorfrom,colorto,max - min + 1,tags[i] - min + 1);
	}

	i = 0;
	$(' > a',this).each(function(){
		
		$(this).css("font-size",fontsize[i] + "px");
		
		$(this).css("color",fontcolor[i]);
		
		$(this).mouseover(function(){
			$(this).animate({
				fontSize: fontmax + "px"
			}, 500);
		});
		
		i++;
		
	});


	// 颜色#FF00FF格式转为Array(255,0,255)
	function color2rgb(color){
		var r = parseInt(color.substr(1, 2), 16);
		var g = parseInt(color.substr(3, 2), 16);
		var b = parseInt(color.substr(5, 2), 16);
		return new Array(r, g, b);
	}
	// 颜色Array(255,0,255)格式转为#FF00FF
	function rgb2color(rgb) {
		var s = "#";
		for (var i = 0; i < 3; i++) {
			var c = Math.round(rgb[i]).toString(16);
			if (c.length == 1)
			c = '0' + c;
			s += c;
		}
		return s.toUpperCase();
	}

	function color(from,to,all,step){
		var Gradient = new Array(3);
		var A = color2rgb(from);
		var B = color2rgb(to);
		for (var i = 0; i < 3; i++) {
			Gradient[i] = A[i] + (B[i]-A[i]) / all * step;
		}
		return rgb2color(Gradient);
	}
}

})(jQuery);
```

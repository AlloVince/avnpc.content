---
title: "图片大小自适应之JQuery版"
date: "2008-05-29 15:48:49"
slug: "image_size_auto_limit"
published: true
author: "AlloVince"
legacy_id: 93
comment_status: "closed"
comments: false
tags:
  - "javascript"
  - "jQuery"
  - "REST"
  - "UI"
  - "YD的程序员葛阁"
---
图片撑破容器是很煞风景的事情，不牵扯到后台生成缩略图的解决方法是在前端对于撑破容器的图片，指定width="100%"，则图片会自动适应父容器的宽度。虽然难看了一点，但不失为一个经济且花费小的方法。

用JS简单的实现思路就是遍历所有页面图片，对于宽度大于父元素的，添加width属性，当然根据需要还可以加上Lightbox之类的特效。

```js
$("img").each(function(){
	if($(this).width() > $(this).parent().width()) {
		$(this).width("100%");
	}
});
```

说了这么多代码只有3行而已。其实对于REST理论，貌似在目前的Web应用中最受惠的可能就是图片这一块了，试想同一个url，如果能对带有不同的header请求返回缩略图，图片描述，图片本身，作者和版权信息，那么图片相关的应用将简洁到一个令人心旷神怡的境界。可惜现在的REST都只是框架级别的应用，对比传统MVC也没有太明显的优势，或许等服务器端进行一定程度的改善之后，REST才能大行其道吧。

话说俺人生的第二个本命年真是多灾多难的一年啊……

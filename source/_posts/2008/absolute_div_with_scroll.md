---
title: "用JQuery实现最简单的随屏幕滚动的层"
date: "2008-11-25 11:16:09"
slug: "absolute_div_with_scroll"
published: true
author: "AlloVince"
legacy_id: 120
comment_status: "closed"
comments: false
tags:
  - "javascript"
  - "jQuery"
  - "Scroll"
  - "YD的程序员葛阁"
---
随屏幕滚动的层是经常能见到的页面特效，貌似还看到有相关的JQuery插件，其实用JQuery一句代码足矣。

首先需要一个绝对定位的元素

```xml
<div id="test" style="position:absolute;">test</div>
```

实现的原理很简单，随页面滚动（Scroll）事件动态设置元素的css top值。

比如定位在页面顶端

```js
$(window).scroll(function(){
	$('#test').css('top', $(document).scrollTop());
});
```

定位在页面底端

```js
$(window).scroll(function(){
	$('#test').css('top', $(document).scrollTop() + $(window).height() - $('#test').height());
});
```

定位在其他位置可以根据具体情况+-top调整即可。

可以参看[DEMO](lab/scroll_div/index.html)

不过固定一个层的最好方法还是用css的position:fixed;。此为后话。

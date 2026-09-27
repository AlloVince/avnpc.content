---
title: "轻量级的Accordion With JQuery"
date: "2008-05-30 16:49:56"
slug: "simple_accordion_with_jquery"
published: true
author: "AlloVince"
legacy_id: 94
comment_status: "closed"
comments: false
tags:
  - "Accordion"
  - "javascript"
  - "jQuery"
  - "UI"
  - "YD的程序员葛阁"
---
Jquery的UI中是有Accordion的，200余行代码配合themes带来的负担，让使用前不可不慎重考虑。

仔细分析一下Accordion的动作其实很简单，顺序排列的N个元素，页面载入时只显示其中一个，点击某个元素时，展开当前元素并隐蔽其他。

那么配合JQuery强大的选择器，完全可以用几行代码打造自己的轻量级Accordion。

例如如下结构的UL列表

```xml
<ul id="accordion">
	<li>
		<span>List Title</span>
		<ul>
		<li>List</li>
		<li>List</li>
		</ul>
	</li>
	<li>
		<span>List Title2</span>
		<ul>
		<li>List</li>
		<li>List</li>
		</ul>
	</li>
</ul>
```

对于这个列表进行操作：

```js
//页面载入时隐蔽除第一个元素外所有元素
$("#accordion > li > span + *:not(:first)").hide();
//对所有元素的标题绑定点击动作
$('#accordion > li > span').click(function(){
	$(this).parent().parent().each(function(){
		//隐蔽所有元素
		$("> li > span + *",this).slideUp();
	});
	//展开当前点击的元素
	$("+ *",this).slideDown();
});
```

总共不过7行，当然选择对象部分需要根据自己的情况灵活指定。

具体可以参看[DEMO](lab/accordion/demo.html)

PS:ASUS Eee，华硕又见华硕，难道我是ASUS控么- -|||

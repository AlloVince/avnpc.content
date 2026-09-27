---
title: "在Javascript中让单击Click和Double Click双击事件各自独立"
date: "2008-12-12 21:23:50"
slug: "independent_click_and_dblclick_event_in_javascript_with_jquery"
published: true
author: "AlloVince"
legacy_id: 122
comment_status: "closed"
comments: false
tags:
  - "click"
  - "dblclick"
  - "event"
  - "javascript"
  - "jQuery"
  - "YD的程序员葛阁"
  - "事件"
---
在Javascript中，如果对一个元素同时绑定Click和Double Click事件，在进行Double Click动作时，程序在响应一个双击事件的同时，还会响应两个单击事件。从原理上来说这应该是正确的，但在开发中往往需要对双击和单击做不同的处理（比如常见的单击选中，双击执行），单双击事件的混杂响应就带来了一定的困扰。本次的目的就是让单击和双击事件各自独立不互相干扰。

首先我们需要弄清楚，浏览器在进行单击和双击时，究竟向DOM元素传递了什么，也就是说Javascript究竟靠什么来区别单击和双击。明白了原理，才好对症下药。

为了简单说明，以JQuery为例，编写测试代码如下：

```js
$("#test").click(function(e){
	var res = '';
	for(var i in e) {
		res += i + ' : ' + e[i] + "\n";
	}
	alert(res);
});
```

来看看单击之后的结果

**IE**

```js
originalEvent : [object]
undefined : undefined
which : undefined
wheelDelta : 0
view : undefined
type : click
toElement : null
timeStamp : 1229092321548
target : [object]
srcElement : [object]
shiftKey : false
screenY : 157
screenX : 140
relatedTarget : undefined
relatedNode : undefined
prevValue : undefined
pageY : 35
pageX : 138
originalTarget : undefined
newValue : undefined
metaKey : undefined
keyCode : 0
handler : function(e){}
fromElement : null
eventPhase : undefined
detail : undefined
data : undefined
currentTarget : undefined
ctrlKey : false
clientY : 37
clientX : 140
charCode : undefined
cancelable : undefined
button : 0
bubbles : undefined
attrName : undefined
attrChange : undefined
jQuery1229092320454 : true
preventDefault : function(){if(d.preventDefault)d.preventDefault();d.returnValue=false}
stopPropagation : function(){if(d.stopPropagation)d.stopPropagation();d.cancelBubble=true}
```

**FF**

```js
originalEvent : [object MouseEvent]
undefined : undefined
which : 1
wheelDelta : undefined
view : [object Window]
type : click
toElement : undefined
timeStamp : 11258562
target : [object HTMLDivElement]
srcElement : undefined
shiftKey : false
screenY : 154
screenX : 124
relatedTarget : null
relatedNode : undefined
prevValue : undefined
pageY : 46
pageX : 124
originalTarget : [object HTMLDivElement]
newValue : undefined
metaKey : false
keyCode : undefined
handler : function (e) {}
fromElement : undefined
eventPhase : 2
detail : 1
data : undefined
currentTarget : [object HTMLDivElement]
ctrlKey : false
clientY : 46
clientX : 124
charCode : undefined
cancelable : true
button : 0
bubbles : true
attrName : undefined
attrChange : undefined
jQuery1229092311220 : true
preventDefault : function () {if (d.preventDefault) {d.preventDefault();}d.returnValue = false;}
stopPropagation : function () {if (d.stopPropagation) {d.stopPropagation();}d.cancelBubble = true;}
```

将测试代码事件改为dblclick之后可以很简单的得到双击的结果

**IE**

```js
originalEvent : [object]
undefined : undefined
which : undefined
wheelDelta : 0
view : undefined
type : dblclick
toElement : null
timeStamp : 1229094138735
target : [object]
srcElement : [object]
shiftKey : false
screenY : 173
screenX : 126
relatedTarget : undefined
relatedNode : undefined
prevValue : undefined
pageY : 51
pageX : 124
originalTarget : undefined
newValue : undefined
metaKey : undefined
keyCode : 0
handler : function(e){}
fromElement : null
eventPhase : undefined
detail : undefined
data : undefined
currentTarget : undefined
ctrlKey : false
clientY : 53
clientX : 126
charCode : undefined
cancelable : undefined
button : 0
bubbles : undefined
attrName : undefined
attrChange : undefined
jQuery1229094137079 : true
preventDefault : function(){if(d.preventDefault)d.preventDefault();d.returnValue=false}
stopPropagation : function(){if(d.stopPropagation)d.stopPropagation();d.cancelBubble=true}
```

**FF**

```js
originalEvent : [object MouseEvent]
undefined : undefined
which : 1
wheelDelta : undefined
view : [object Window]
type : dblclick
toElement : undefined
timeStamp : 1229094216813
target : [object HTMLDivElement]
srcElement : undefined
shiftKey : false
screenY : 141
screenX : 137
relatedTarget : null
relatedNode : undefined
prevValue : undefined
pageY : 33
pageX : 137
originalTarget : [object HTMLDivElement]
newValue : undefined
metaKey : false
keyCode : undefined
handler : function (e) {}
fromElement : undefined
eventPhase : 2
detail : 2
data : undefined
currentTarget : [object HTMLDivElement]
ctrlKey : false
clientY : 33
clientX : 137
charCode : undefined
cancelable : true
button : 0
bubbles : true
attrName : undefined
attrChange : undefined
jQuery1229094214813 : true
preventDefault:function(){if(d.preventDefault){d.preventDefault();}d.returnValue=false;}
stopPropagation:function(){if(d.stopPropagation){d.stopPropagation();}d.cancelBubble=true;}
```

需要注意的是type和detail这两项，type标记了事件的类型，detail则是Firefox独有的属性，对单击进行了计数，双击时第一次为1，第二次为2。

即是说在FF中，还是有办法区别两次单击动作的，但无论单击双击，detail都先从1开始计数，所以想从事件自带的属性并不能达到独立事件的目的。

这里有[一篇关于单双击问题](http://bbs.jquery.org.cn/thread-3060-1-10.html)的解决，提供了一种思路，简单说是将第一次单击延时执行，如果在延时期间出现第二次单击，则取消第一次单击的执行。

按照上述办法我们可以模拟出近似效果，可以参看[DEMO](lab/fixclick/demo.html)，除了单击时有延时之外，基本实现了事件的独立。

为了方便，做成[JQuery插件](http://plugins.jquery.com/project/FixClick)，有兴趣的同学可以自取所需。

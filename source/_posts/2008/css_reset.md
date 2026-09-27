---
title: "CSS Reset总结"
date: "2008-11-21 16:46:26"
slug: "css_reset"
published: false
author: "AlloVince"
legacy_id: 119
comment_status: "closed"
comments: false
tags:
  - "css"
  - "YD的程序员葛阁"
  - "YUI"
---
Css Reset即是对浏览器的默认Css规则进行初始化，保证不同浏览器下Css的一致性。

目前有很多现成的Css Reset方案，不过都存在一定差异。毕竟Css Reset要经过大量不同浏览器的实地验证才能获得结论，所以对时下的这些Css Reset方案，也无法简单的判定好坏，只是列举出来对比如下。

考察对象是使用率比较高的三个：

- [YUI](http://developer.yahoo.com/yui/reset/)
- [Erik Meyer’s CSS Reset](http://meyerweb.com/eric/tools/css/reset/index.html)
- Condensed Meyer Reset

| 内容\对象 | YUI | Condensed | Erik Meyer’s |
| :-: | :-: | :-: | :-: |
| margin/padding 重置范围 | 常用元素 | 常用元素 | 几乎全部 |
| border 重置范围 | 少量元素 | 少量元素 | 几乎全部 |
| font-size 重置范围 | h1\~h6 | h1\~h6 | 几乎全部 |
| font-weight 重置范围 | h1\~h6 + 部分元素 | h1\~h6 | × |
| outline/vertical-align/background 重置范围 | × | × | ◎ |
| ol, ul list-style | ◎ | ◎ | ◎ |
| table　border-collapse/border-spacing | ◎ | ◎ | ◎ |
| :focus | × | × | ◎ |
| blockquote:before, blockquote:after | × | × | ◎ |
| q:before, q:after | ◎ | ◎ | ◎ |
| ins/del | × | × | ◎ |
| caption,th | ◎ | ◎ | × |
| body line-height | × | × | ◎ |

可以看得出Erik方案是不管三七二十一，对差不多所有的Html元素都做了重置，YUI则相对有选择性一些，Condensed方案比较折中。而对于几个独有的重置，则最好抱着宁可信其有的归纳整理比较好。

放上自己最后整理使用的版本

```css
/**
* CSS Reset
* by YUI http://developer.yahoo.com/yui/reset/ and Erik Meyer http://meyerweb.com/eric/tools/css/reset/index.html
* Pack up by AlloVince http://allo.ave7.net/css_reset
**/
body,div,dl,dt,dd,ul,ol,li,h1,h2,h3,h4,h5,h6,pre,form,fieldset,input,textarea,p,blockquote,th,td {
	margin:0;
	padding:0;
	outline: 0;
	vertical-align: baseline;
	background: transparent;
}
body {
	line-height: 1;
}
h1,h2,h3,h4,h5,h6,address,caption,cite,code,dfn,em,strong,th,var {
	font-style:normal;
	font-size:100%;
	font-weight:normal;
}
table {
	border-collapse:collapse;
	border-spacing:0;
}
ol,ul {
	list-style:none;
}
:focus {
	outline: 0;
}
blockquote:before, blockquote:after,q:before, q:after {
	content: '';
	content: none;
}
fieldset,img,abbr,acronym {
	border:0;
}
ins {
	text-decoration: none;
}
del {
	text-decoration: line-through;
}
caption,th {
	text-align:left;
}
```

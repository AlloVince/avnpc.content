---
title: "JQuery SyntaxHighlighter plugin"
date: "2008-04-21 16:34:56"
slug: "JQuery_with_SyntaxHighlighter"
published: true
author: "AlloVince"
legacy_id: 89
comment_status: "closed"
comments: false
tags:
  - "javascript"
  - "jQuery"
  - "SyntaxHighlighter"
  - "YD的程序员葛阁"
---
**目录** [折叠]

1. [Features](#toc_01848200)
2. [Demo](#toc_01859900)
3. [Useage](#toc_01868500)
4. [Downloads](#toc_01878900)

[SyntaxHighlighter](http://code.google.com/p/syntaxhighlighter/)是一个出色的语法高亮库，但实际使用，尤其是大量Js文件的包含，仍有不便之处。另外在W3C规范中，Pre元素是不能使用name属性的。这里通过JQuery动态加载所需的SyntaxHighlighter文件，将使用过程最简化。

#### Features

Load js files automatic,Simple to use SyntaxHighlighter.

You can use this plugin with only 1 line code :

```js
$.SyntaxHighlighter('./SyntaxHighlighter/');
```

to instead of this :

```xml
<script language="javascript" src="SyntaxHighlighter/Scripts/shCore.js"></script>
<script language="javascript" src="SyntaxHighlighter/Scripts/shBrushCSharp.js"></script>
<script language="javascript" src="SyntaxHighlighter/Scripts/shBrushXml.js"></script>
<script language="javascript" src="SyntaxHighlighter/Scripts/shBrushCpp.js"></script>
<script language="javascript" src="SyntaxHighlighter/Scripts/shBrushCss.js"></script>
<script language="javascript" src="SyntaxHighlighter/Scripts/shBrushDelphi.js"></script>
<script language="javascript" src="SyntaxHighlighter/Scripts/shBrushJava.js"></script>
<script language="javascript" src="SyntaxHighlighter/Scripts/shBrushJScript.js"></script>
<script language="javascript" src="SyntaxHighlighter/Scripts/shBrushPhp.js"></script>
<script language="javascript" src="SyntaxHighlighter/Scripts/shBrushPython.js"></script>
<script language="javascript" src="SyntaxHighlighter/Scripts/shBrushRuby.js"></script>
<script language="javascript" src="SyntaxHighlighter/Scripts/shBrushSql.js"></script>
<script language="javascript" src="SyntaxHighlighter/Scripts/shBrushVb.js"></script>
```

```js
dp.SyntaxHighlighter.ClipboardSwf = 'SyntaxHighlighter/Scripts/clipboard.swf';
dp.SyntaxHighlighter.HighlightAll('code');
```

#### Demo

Here is the [Demo link](lab/highlighter/demo.html), compare to the [page without plugin](lab/highlighter/without_plugin.html).

#### Useage

```js
$.SyntaxHighlighter('./SyntaxHighlighter/'); //the path of SyntaxHighlighter files
```

```xml
<pre class="js">
... some code here ...
</pre>
```

Advance Options,and the [Advance Demo](lab/highlighter/advance_demo.html).

```js
	var option = {
		dir:'./SyntaxHighlighter/', //required. path of SyntaxHighlighter
		/**
		 * Set SyntaxHighlighter default options on this page
		 * You can get more info at :
		 * http://code.google.com/p/syntaxhighlighter/wiki/HighlightAll
		*/
		name:"SyntaxHighlighter", //optional, default:"SyntaxHighlighter"
		showGutter:true, //optional, default:true
		showControls: false, //optional, default:false
		collapseAll:false, //optional, default:false
		firstLine : 1, //optional, default:false
		showColumns:false, //optional, default:false
		/**
		 * Options of this Plugin
		*/
		apptoall:false, //optional, default:true. enable default options to all elements instead of elements self option.
		autofind:false, //optional, default:true.
		//auto enable highlighter to <pre> and <textarea> elements with 'class' attribute on this page whether with 'name'.
		jspath:'./SyntaxHighlighter/Scripts/', //optional, default:dir + 'Scripts/'. path of SyntaxHighlighter Js files
		csspath:'./SyntaxHighlighter/Styles/', //optional, default:dir + 'Styles/'. path of SyntaxHighlighter Css files
		swfpath:'./SyntaxHighlighter/Scripts/' //optional, default:dir + 'Scripts/' path of SyntaxHighlighter clipboard.swf file
	};
	$.SyntaxHighlighter(option);
```

#### Downloads

[Download Link](http://cid-01e48df64f8bd957.skydrive.live.com/embedrowdetail.aspx/Source/highlighter.7z)

Or get this plugin in [JQuery Plugin Site](http://plugins.jquery.com/project/Jlighter)

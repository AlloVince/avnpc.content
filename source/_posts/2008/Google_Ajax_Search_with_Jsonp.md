---
title: "使用Google Ajax Search with Jsonp 构建纯Javascript站内搜索"
date: "2008-08-07 14:32:57"
slug: "Google_Ajax_Search_with_Jsonp"
published: true
author: "AlloVince"
legacy_id: 109
comment_status: "closed"
comments: false
tags:
  - "Google"
  - "javascript"
  - "jQuery"
  - "JSONP"
  - "Search"
  - "YD的程序员葛阁"
---
**目录** [折叠]

1. [最简单的Google Jsonp搜索](#toc_16260300)
2. [最简单的Google Jsonp本地搜索](#toc_16280800)
3. [更多问题](#toc_16291300)

利用[Google AJAX Search](http://code.google.com/apis/ajaxsearch/)可以很快的建立起一个本地搜索，关于构建的方法，Google docs里已经有详细的讲述，也并非本文的重点。

利用Google的函数库，不可避免的要嵌入远端文件，数据样式也是经过处理後的。有些时候，由于种种条件限制，我们需要更加灵活轻便的解决方案，这时候就需要借助Google Ajax Search中的[Jsonp](http://www.json.org/)协议。

#### 最简单的Google Jsonp搜索

在[Flash and other Non-Javascript Environments](http://code.google.com/apis/ajaxsearch/documentation/#fonje)一节中，可以了解到，如果对

http://ajax.googleapis.com/ajax/services/search/web?v=1.0

发送Get请求，就能返回相应的Jsonp格式数据，比起标准API来，这里的数据不携带任何冗余信息，可以很方便的处理和调用。例如使用JQuery，短短几行代码就可以实现Ajax的搜索引擎。

首先当然要有一个搜索表单

```xml
<form class="googlesearch" action="http://www.google.com/search" onsubmit="search(this);return false;">
	<div id="search_box">
			<input id="searchvalue" value="" name="q" />
			<input id="searchsubmit" type="submit" value="搜索" />
	</div>
</form>
```

然后是表单提交时调用的Js函数

```js
function search(form){
	$.ajax({
		url:'http://ajax.googleapis.com/ajax/services/search/web?v=1.0&q=' + encodeURIComponent(form.q.value),
		dataType : 'jsonp',
		success: function(json){
			var res = '';
			for(var i in json.responseData.results){
				for(var j in json.responseData.results[i]){
					res = res + j + ':' + json.responseData.results[i][j] + "\n";
				}
			}
			alert(res);
		}
	});
}
```

可以查看做好的[Demo](lab/google_ajax_search_json/google_ajax_search_with_jsonp.html)，点击搜索，会弹出google的返回的搜索结果。

#### 最简单的Google Jsonp本地搜索

仅仅这些，还不能满足一个构成本地搜索的条件。在标准API中，可以通过

```js
var siteSearch = new GwebSearch();
siteSearch.setUserDefinedLabel("allo.ave7.net");
siteSearch.setUserDefinedClassSuffix("siteSearch");
```

来实现对特定域名"allo.ave7.net"的检索，jsonp中怎么实现相同的功能？只能通过增加查询关键词"site:allo.ave7.net"来达到目的。即在上文的url部分改为

```js
url:'http://ajax.googleapis.com/ajax/services/search/web?v=1.0&q=' + encodeURIComponent(form.q.value) + '+site%3Aallo.ave7.net',
```

来看看最简单的本地搜索：[Demo](lab/google_ajax_search_json/google_local_search_with_jsonp.html)。在这个Demo里，所有检索出的数据都限定在域名allo.ave7.net里。

#### 更多问题

就像前面的本地搜索一样，开发往往要需要很多可以设置的参数，比起标准API来，Jsonp只能通过url传达设定，能预设的参数不够丰富。具体的参数可以参看[Flash and other Non-Javascript Environments](http://code.google.com/intl/en/apis/ajaxsearch/documentation/reference.html#_intro_fonje)一节，需要注意的一些问题有：

- 分页的实现：Jsonp只返回前4页以及总记录数，事实上如果发送第四页以后的请求，仍然能够获得正确的回应，只是分页函数要自己写了。
- 每页的条目数：这个设定比较苛刻，只能通过rsz=small/large来获得每页4条/8条的数据，可能是Jsonp最不方便之处了。
- 结果的过滤：Google Web搜索会自动过滤一些可能重复的项目，留意观察一下google的搜索参数，是通过filter=0/1开关的。但在Jsonp的响应中，对第一页的请求，过滤是关闭的，第二页以后则又自动打开了。不算是大问题，但会让分页的现实变得很奇怪。虽然文档中提到可以通过safe=active/moderate/off来改变过滤，但写入这个参数后并没有影响到检索结果。希望知情人士告知问题所在。

最后将allo.ave7.net的本地搜索单独抽出做成[Demo](lab/google_ajax_search_json/google_search_with_jsonp_advance.html)，也可以下载本次的[全部源代码](http://cid-01e48df64f8bd957.skydrive.live.com/embedrowdetail.aspx/Source/searchjson.7z)。

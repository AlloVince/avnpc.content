---
title: "vBulletin with JSONP（Javascript跨域问题及基于vbb单点登录浅谈）"
date: "2008-04-24 12:35:53"
slug: "vBulletin_with_JSONP"
published: true
author: "AlloVince"
legacy_id: 91
comment_status: "closed"
comments: false
tags:
  - "javascript"
  - "jQuery"
  - "JSONP"
  - "vBulletin"
  - "YD的程序员葛阁"
  - "单点登录"
---
出于浏览器安全考虑，javascript是不能跨域访问的，即是说js使用XMLHTTPRequest的访问对象只限于同一域名，统一端口，统一协议下的数据。如果未加任何处理，js会报错nsresult: "0x8057001e (NS_ERROR_XPC_JS_THREW_STRING)"。

但事实上大型系统往往需要跨越多台服务器传递数据，那么如何解决这一问题，时下有如下的若干解决方法：

- 隐蔽iFrame
  - 实现成本：低
  - 被访问端控制权限：不需要
  - 实现方法：在页面上创建一个不可见的iframe，将所要访问的页面包含进来，然后通过操作iframe来获得所需数据。实际上这种解决方法已经和钓鱼网站没什么区别了，只不过是非恶意的而已。除非是做数据采集，否则不推荐。
- document.domain
  - 实现成本：中
  - 被访问端控制权限：不需要
  - 实现方法：假如要访问ave7.net的数据，在代码开始时加入一行  
    document.domain = 'ave7.net';  
    即可。这大概是js本身提供的解决方案，但实用性并不高，尤其是多浏览器对document.domain的支持参差不齐，至今也没有一个像样的案例出现，不推荐。
- 包含远程文件
  - 实现成本：中
  - 被访问端控制权限：需要
  - 实现方法：假如要访问ave7.net的数据，编写相应xxx.js文件上传至ave7.net，在本地使用 `<script type="text/javascript" src="http://ave7.net/xxx.js"></script>`将访问对象包含进来即可。这样等于完全绕开了ajax 访问，是一个较为实用的解决方案。缺点也很明显，需要两端都有控制权限，代码被迫分离给维护带来了不便，另外无法像Ajax一样判断程序执行到那个阶段，如果远程服务器暂时无法访问，则可能产生致命性错误。
- JSONP
  - 实现成本：高
  - 被访问端控制权限：需要

JSONP是JSON的一个子集，目前还是非官方的协议。形如

```js
callback({jsondata});
```

即是把json数据用一对圆括号括住，前面加上回调函数实现，目前像jquery,dojo等js库都提供了jsonp的支持。

JSONP虽然是简单的通讯协议，但已经实现了很好的接口定义，虽然被访问段需要重新编写接口，但如果是比较复杂的程序应用，就显得十分有必要了。

以我的Blog动态调用vBulletin论坛信息实现简单单点登录为例。通信过程如下图所示：

1. Blog发起AJAX请求，将Session传递给vBulletin
2. vBulletin根据用户cookie获得用户信息UserInfo，将获得的Session和用户信息通过一个加密函数functionA生成一个Hash值HashA
3. vBulletin将HashA和UserInfo回传给Blog
4. Blog将接受到的UserInfo以同样的函数functionA生成HashB
5. 如果HashA与HashB相等，则作为用户登录。

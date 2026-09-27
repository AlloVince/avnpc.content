---
title: "官方同步插件Weave0.2 for Firefox3试用简评"
date: "2008-07-03 13:30:28"
slug: "Mozilla_Weave"
published: true
author: "AlloVince"
legacy_id: 101
comment_status: "closed"
comments: false
tags:
  - "Firefox"
  - "Weave"
  - "YD的程序员葛阁"
---
[Weave](https://services.mozilla.com/)是Mozilla官方推出的浏览器同步项目，目前Firefox的同步插件[Google Browser Sync](http://www.google.com/tools/firefox/browsersync/)已经停止了开发，所以Firefox3.0中尚没有可用的同步插件，而Weave0.2的推出给FF3带来了一点希望。

安装，重启，试用……一个小时后我对Weave0.2的表现只有四个字可以形容：“惨不忍睹”。

首先是不友好，账户注册完毕到数据上传是完全没有问题的，但如果要下载同步数据，需要从邮件激活Weave服务，否则会一直提示连接服务器超时。基本的提示信息都起不到相应的向导作用，很难相信这是面向普通用户的产品。

然后是速度，Weave的同步不是像Google Browser Sync那样即时通信，而是退出FF后执行，而服务器又慢到不能忍，每次退出都要停顿30秒至数分钟，最可怕的是如果强行中断同步则会驻留一个 Firefox进程在内存中，真怀疑这不是官方插件，而是官方病毒- -|||

对于真正的同步功能也很难让人满意，用作测试的数据是1000个左右的书签，但同步过程中浏览器却严重拖慢，连正常的浏览都无法进行。不知道是服务器速度影响还是插件本身缺陷。

虽然Weave的野心很大，在预定开发的功能里还能看到插件同步等等让人浮想联翩的词汇，但在一个勉强能用的版本出现之前，需要同步功能的同学还是老老实实用FF2.0+Google Browser Sync吧

另外补充一个Firefox3 css hack之JQuery版

```js
$.browser.mozilla && parseFloat($.browser.version) > 1.8
```

用js获得的FF3的版本并不是软件开发版本3.0，而是Gecko内核版本1.9，很有意思

---
title: "例行汇报"
date: "2009-01-13 16:33:39"
slug: "regular_reporting"
published: true
author: "AlloVince"
legacy_id: 123
comment_status: "closed"
comments: false
tags:
  - "avplayer"
  - "YY的每一天"
  - "音乐盒"
---
终于忍不住为时空之滨做了[新的临时过渡画面](http://tsost.ave7.net/)，当然因为偷懒，很多东西都拿现成的来用，所以这只是一个集成了Google Reader、Twitter、Delicious服务的前端聚合，不过总算是看起来舒服了不少。

音乐盒进度报告，后台部分仍在缓慢的建设中，暂时决定用Rails搭一个简单的构架出来，放出DB部分的初步E-R图

[![](https://lh4.googleusercontent.com/-BRif8RCi3HE/SUnmzkYI6EI/AAAAAAAABww/WeX3vChRa4E/s288/db.jpg)](https://picasaweb.google.com/lh/photo/iiqPtqPswauNXwlhowMSl9MTjNZETYmyPJy0liipFm0?feat=embedwebsite)

前端部分则要根据[AvPlayer](http://code.google.com/p/tsostplayer/)的开发情况决定。

AvPlayer已经华丽的发布了0.10版，也有了几个实质性的Demo，比如作为迷你播放器结合[swfobject](http://code.google.com/p/swfobject/)实现[一行代码的简单嵌入](lab/avplayer/Demos/player_js_write.html)，以及[完全用Js操作播放器](lab/avplayer/Demos/player_javascript.html)等等，虽然还处于项目的最初阶段，不过拿来用用已经不成问题了。

整体的前端搭建将是对自己这一年来的一个整理总结，给自己的要求是：

- 通过W3C Strict的语义化页面
- 样式布局与内容解耦
- 用更好的布局和CSS来代替滥用的css Hack和js补正
- 无侵入和模块化的Js
- 完善的Ajax回退问题以及部分刷新后DOM事件重新绑定问题的解决方案
- 通过RESTful来实现更简化的前端数据获取

发现果然还是很享受这种没有什么回报，但是可以自由发挥的感觉，真是遗憾哪- -。

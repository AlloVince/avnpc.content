---
title: "基于ActionScript3的Flash Mp3 Player做成记录(序)"
date: "2008-09-04 16:41:38"
slug: "Flash_Mp3_Player_with_ActionScript3_0"
published: true
author: "AlloVince"
legacy_id: 111
comment_status: "closed"
comments: false
tags:
  - "ActionScript"
  - "Flash"
  - "Mp3 Player"
  - "YD的程序员葛阁"
---
**目录** [折叠]

1. [罗嗦的前言](#toc_56728600)
   1. [JW FLV MEDIA PLAYER](#toc_56734800)
   2. [Dewplayer](#toc_56743600)
   3. [Flowplayer](#toc_56751100)
   4. [WordPress Audio Player](#toc_56756600)
   5. [Yahoo! Media Player](#toc_56771300)

1. [序言](blog/view/111)
2. [Hello Player](blog/view/112)
3. [播放、暂停与停止](blog/view/113)
4. [Demo及项目页](blog/view/117)

#### 罗嗦的前言

其实想做一个自主开发的Web Mp3 Player已经有很久了。作为一个懒人，如果产生了要做什么的想法，那最大原因一定是现有的东西无法满足需要。

或许只是程序员蜀黍的劣根性使然，目前的Web Mp3 Player项目中并不乏优秀的作品，但在一一试用之后还是决定自己开发，不过还是稍作列举：

##### JW FLV MEDIA PLAYER

[JW FLV MEDIA PLAYER](http://www.jeroenwijering.com/?item=JW_FLV_Player)是历史很悠久的项目了，从最初的一个简单的Mp3播放器不断改进到现在的音乐图片视频通吃的全能型Media Player，作者一直在努力。

基本上已经是比较完善的作品，可以自己制作皮肤，Js的API也比较丰富，不过如果用做扩展开发的话，不开源这一条就可以枪毙了。

##### Dewplayer

采用[Dewplayer](http://www.alsacreations.fr/dewplayer)的网站非常多，简洁的外观，5KB的体积非常适合嵌入各类网页和Blog。现在最新版也提供了简单的Js接口。

不过很遗憾这也不是开源产品，作者还准备了Pro版本赚钱，功能简单，用用就好。

##### Flowplayer

[Flowplayer](http://flowplayer.org/)也是屡次出现在开源项目推荐榜上的名字，我了解不多，也许是因为Licence的问题，应用没有想象的广泛。

##### WordPress Audio Player

看名字也知道[WordPress Audio Player](http://wpaudioplayer.com/)是专为WP打造的，不过即便如此，将播放器部分独立出来也是很优秀的作品。本Blog一直以来也是采用着Audio Player。

非常简洁的设计，所有部件都是用AS脚本渲染生成，颜色可以在客户端做自定义。很巧妙的设计了JS接口：如果页面内有复数个Audio Player时，播放其中一个，其他正在播放的都会自动关闭。

##### Yahoo! Media Player

[Yahoo!出品](http://mediaplayer.yahoo.com/)，强大令人赞叹的作品，强大之处在于Yahoo! Media Player同时利用了Windows Media Player的接口和Flash技术，极大限度的保证了使用者无需额外安装插件。使用方面也是简单到了只需要引入Js文件后就可以自动侦测页面的所有指向Mp3的超链接生成播放器。

但也正是由于这样过度的封装，完全没有自定义和二次开发的余地。Yahoo! Media Player更像是观赏品，实用价值不高。

---

啰啰嗦嗦这么多，以上这些项目都不能让我满意的原因有三：

- 暴露Mp3地址，曲目管理功能弱
- 不支持歌词功能
- 很难进行二次开发

所以从下一次日志开始，将记录一个基于ActionScript3的Flash Mp3 Player的做成过程。暂定项目名「[AVplayer](http://code.google.com/p/tsostplayer/)」 -\_,-

PS。订阅的RSS快被铺天盖地的Chrome新闻淹没了，怎么说Chrome还只是一个试验品，连最基本的CSS渲染都存在错误，占据市场份额更是不着边际的事情，Google还是快点着手修复目前的BUG吧。

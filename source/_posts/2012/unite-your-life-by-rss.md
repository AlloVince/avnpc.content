---
published: true
date: '2012-06-04 13:57:41'
tags: []
author: AlloVince
title: 通过 RSS 聚合你的生活
---

一直以来都想做一个聚合类的服务，能将我在各种网站上的信息汇聚起来，形成专属于我自己的，独一无二的个人生活记录。我想我的核心需求应该可以归结为三个：

1.  聚合，可以支持绝大多数主流的服务，尤其可以符合国情
2.  个性化，Friendfeed 曾经让我眼前一亮，但是 Friendfeed 仅仅能完成基本的聚合，并不能按照自己的想法任意改造
3.  完全自主，我的数据就是我的，如果依托给第三方平台，那一天平台玩完了我的数据也没有了依托，这是不能允许的

愿景很美好，但是附带的工作量非常可观，所以这个想法还一直停留在初级阶段，见过我 blog 上一个版本的童鞋可以看到这个想法的雏形，那是一个纵向时间轴，按时间顺序简单显示 Twitter 和 Blog 内容。

而最近又忍不住手痒，找到了实现这一想法更为低廉的手段，就有了这样一个只属于我自己的无限瀑布流：

[Life of AlloVince][http://avnpc.com/pages/unite-your-life-by-rss]

原本要聚合第三方服务，访问第三方的 API 是必不可少的，但是作为简化版，最平易近人的解决方案还是通过 RSS 聚合信息。

瀑布流聚集了以下我常用的第三方服务：

-   个人微博公共信息
-   Twitter 信息
-   公共 Google+信息
-   豆瓣公开信息（书/电影/音乐）
-   豆瓣电台红心歌曲
-   Lastfm 收藏曲目
-   Evernote 公开笔记
-   Google Reader 加星条目

实现方法也很简单，将这些信息转为 RSS，然后统一订阅到 Google Reader 即可

其中原生提供 RSS 的有以下服务：

-   Twitter
    <del>RSS 在个人页面上貌似已经看不到了，格式如下，user\_id 为 twitter 用户的数字 ID</del>
    
<del>http://twitter.com/statuses/user_timeline/{user_id}.atom</del>

上面的 Twitter 的 RSS 获得方法已经失效，新的地址可以使用：

```plain
    https://search.twitter.com/search.atom?q=from:UserName
```

将 UserName 替换为你的 Twitter ID

-   豆瓣 RSS，请访问豆瓣个人主页，比如我的豆瓣个人主页
    [http://www.douban.com/people/AlloVince/][]，
    对应的 RSS 为[http://www.douban.com/feed/people/AlloVince/interests][]
-   Lastfm 的 RSS 说明在此[http://cn.last.fm/api/feeds][]，我的 RSS 为：[http://ws.audioscrobbler.com/2.0/user/AlloVince/lovedtracks.rss][]
-   Evernote 公开笔记需要先在 Evernote 中共享笔记本，然后进入在线服务找到对应笔记本即可。遗憾的是 Evernote 提供的 RSS 不是全文输出，也去掉了格式。
-   Google
    Reader 加星条目的 RSS 很隐蔽，格式如下。其中 user\_id 部分是一串数字，在 Google
    Reader 点击加星号的条目后 URL 里即可看到。为了和谐，建议用 https 订阅

```plain
    http://www.google.com/reader/public/atom/user/{user_id}/state/com.google/starred。 
```

其他的信息需要借助一些第三方服务实现：

-   <del>公共微博转 RSS 我使用了[微博 RSS 生成器][]</del>。
-   微博的处理目前有两个转换服务：[weiborss](http://weiborss.com/)和[新浪微博 RSS 订阅
](http://rssgen.sharingmadeeasy.com/)，但是转换的 RSS 效果都不够好。可能还是从 API 最靠谱吧
-   公共 Google+信息需要通过 gplusrss 服务：[http://gplusrss.com/][]

豆瓣 FM 红心歌曲是最复杂的。首先需要有 Firefox +
[Greasemonkey 插件][]，然后安装[豆瓣电台 scrobbler][]，这样可以在豆瓣电台播放时，同步所有曲目到 Lastfm，包括红心曲目。

最后将所有的 RSS 归类到一个文件夹下，就可以通过操作[Google Reader
API][]来获得聚合后的 RSS 了。Google Reader 可以存档所有的历史条目，可以简单实现翻页和查询。

连 RSS 订阅+程序可能总计用了 5 个小时左右，实现成本近乎为 0，不过缺点也不少：

-   只能记录从订阅 RSS 开始时刻起的信息
-   旧的信息无法更新删除
-   支持的服务有限

有独立博客的童鞋可以尝试组合一些 RSS 插件实现类似效果。


  [Life of AlloVince]: /life "http://avnpc.com/life"
  [http://www.douban.com/people/AlloVince/]: http://www.douban.com/people/AlloVince/
    "http://www.douban.com/people/AlloVince/"
  [http://www.douban.com/feed/people/AlloVince/interests]: http://www.douban.com/feed/people/AlloVince/interests
    "http://www.douban.com/feed/people/AlloVince/interests"
  [http://cn.last.fm/api/feeds]: http://cn.last.fm/api/feeds
    "http://cn.last.fm/api/feeds"
  [http://ws.audioscrobbler.com/2.0/user/AlloVince/lovedtracks.rss]: http://ws.audioscrobbler.com/2.0/user/AlloVince/lovedtracks.rss
    "http://ws.audioscrobbler.com/2.0/user/AlloVince/lovedtracks.rss"
  [微博RSS生成器]: http://ishow.sinaapp.com/rss.php
    "http://ishow.sinaapp.com/rss.php"
  [http://gplusrss.com/]: http://gplusrss.com/ "http://gplusrss.com/"
  [Greasemonkey插件]: http://www.greasespot.net/
    "http://www.greasespot.net/"
  [豆瓣电台scrobbler]: http://userscripts.org/scripts/show/98833
    "http://userscripts.org/scripts/show/98833"
  [Google Reader API]: http://code.google.com/p/pyrfeed/wiki/GoogleReaderAPI
    "http://code.google.com/p/pyrfeed/wiki/GoogleReaderAPI"



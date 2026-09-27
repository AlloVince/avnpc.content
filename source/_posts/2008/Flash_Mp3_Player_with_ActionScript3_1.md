---
title: "基于ActionScript3的Flash Mp3 Player做成记录(1)"
date: "2008-09-09 15:13:29"
slug: "Flash_Mp3_Player_with_ActionScript3_1"
published: true
author: "AlloVince"
legacy_id: 112
comment_status: "closed"
comments: false
tags:
  - "ActionScript"
  - "Flash"
  - "Mp3 Player"
  - "YD的程序员葛阁"
---
**目录** [折叠]

1. [都要做些什么](#toc_77418900)
2. [从面向过程开始](#toc_77449600)
3. [总之先发出声音](#toc_77453800)

前情提要：[基于ActionScript3的Flash Mp3 Player做成记录](blog/view/111)

#### 都要做些什么

动手之前，首先要做的是理清楚一个播放器可能包括的功能模块，然后从简到难，一步一步完成。让我们列个单子：

1. 读入声音文件
2. 声音的播放与停止
3. 计时器
4. 文件读取进度条
5. 声音播放进度条
6. 在进度条上可拖拽的游标及改变播放位置
7. 音量与声道控制
8. 读取MP3 ID3 Tag
9. 将信息放入外部文件中
10. 文件列表的播放
11. 搭建外部接口
12. 歌词功能

细节会有很多变化，但大体上是这些基本的功能组成了一个音频播放器。本记录也会按照这个顺序进行。

#### 从面向过程开始

OOP的AS3，为什么要从面向过程开始？其实这只是属于个人习惯，从面向过程开始，可以最短时间内看到做出来的东西，也可以随便修修补补，权当在打草稿。OOP的整理，会放在基本模型搭建完毕之后进行。所以本记录将有很长一段时间停留在面向过程的层面。

#### 总之先发出声音

为了证明上面的话，我们可以用最短的时间先折腾出一点声音来让气氛轻松一点。

用Flash CS3新建并保存一个名为avplayer.fla的文件，同目录下放置名为test.mp3的音频文件。点击F9呼出动作窗口，然后输入以下代码。

```js
import flash.media.Sound;
import flash.net.URLRequest;

var _s:Sound = new Sound();
_s.load(new URLRequest("test.mp3"));
_s.play();
```

然后按ctrl+enter，不出意外的话，现在已经能够听到美妙的音乐了。恩，Mp3 Player并不是很难的东西。

最后放上本次的[代码](http://cid-01e48df64f8bd957.skydrive.live.com/embedrowdetail.aspx/Source/avplayer1.7z)。

---
title: "基于ActionScript3的Flash Mp3 Player做成记录(2) - 播放、暂停与停止"
date: "2008-09-19 16:02:11"
slug: "Flash_Mp3_Player_with_ActionScript3_2"
published: true
author: "AlloVince"
legacy_id: 113
comment_status: "closed"
comments: false
tags:
  - "ActionScript"
  - "Flash"
  - "Mp3 Player"
  - "YD的程序员葛阁"
---
**目录** [折叠]

1. [如何绑定事件](#toc_91774700)
2. [设置全局变量](#toc_91797300)
3. [播放操作](#toc_91823100)
4. [停止与暂停](#toc_91842900)
5. [整合](#toc_91852800)

前情提要：[基于ActionScript3的Flash Mp3 Player做成记录](blog/view/111)

#### 如何绑定事件

播放器最基本的功能莫过于播放，暂停和停止，本次便来完成这三个功能。

在Code之前，必要的UI搭建是不能少的，这里建立了三个按钮并依次命名实例为：

- 播放按钮：buttPlay
- 暂停按钮：buttPause
- 停止按钮：buttStop

当然怎么做Flash按钮并不在本文的讨论之列，这里一笔带过。

有了按钮，很自然的会想到如何在按钮上绑定事件，幸运的是在经历了AS1、AS2的空前混乱之后，AS3.0中终于将事件的绑定统一为唯一的addEventListener方法，除此之外别无他路。例如

```js
buttPlay.addEventListener(MouseEvent.CLICK,onPlay);
```

就是在buttPlay按钮上绑定了一个名"onPlay"的函数，这个函数在鼠标点击时触发。这种绑定类似于Javascript的addEventListener，写过Js的同学应该很容易接受。

所以现在我们只需要对每次按钮绑定事件，然后分别编写每个事件的函数，就实现了最简单的与用户的UI交互：

```js
function onPlay(event:MouseEvent):void {}
function onPause(event:MouseEvent):void {}
function onStop(event:MouseEvent):void {}

buttPlay.addEventListener(MouseEvent.CLICK,onPlay);
buttPause.addEventListener(MouseEvent.CLICK,onPause);
buttStop.addEventListener(MouseEvent.CLICK,onStop);
```

#### 设置全局变量

现在可以正式开始了么？很遗憾，在深入到每个事件之前，我们首先要考虑怎么设置全局变量。这是由于addEventListener式的事件绑定虽然便于管理，但可能实际一操作就会发现，要向绑定的函数中传递参数，解决的方法只是在函数外设置全局变量。

在这里我们设置了4个全局变量：

```js
//当前请求的Mp3文件
var _req:URLRequest;
//声音基本类
var _sound:Sound = new Sound();
//声音播放控制
var _channel:SoundChannel;

var _status = {
	firstClick:false,
	loadComplete:false,
	isPlaying:false,
	pausePosition:0,
	length:0,
	played:0,
	volume:0.7
};
```

这里定义的_status包含了正在播放声音的所有必要信息，所有的操作通过这个变量联系起来，是播放器控制的核心：

- firstClick：是否是第一次点击，第一次点击需要读入文件
- loadComplete：声音文件是否已读取完毕
- isPlaying：是否正在播放
- pausePosition：播放暂停的位置
- length：声音文件的长度
- played：已经播放的长度
- volume：音量大小

#### 播放操作

现在终于可以进入到第一个事件里了。来考虑一下当点击播放按钮之后，播放器应该做些什么：

- 如果声音已经开始播放，不做任何操作
- 如果是第一次点击，那么文件还未读入，首先要将MP3文件读入
- 读入之后开始播放，将状态firstClick和isPlaying置为true
- 为了知道是否已经播放完毕，还需要对播放完成（SOUND_COMPLETE）这个时间进行监听

转换成代码就是

```js
function actionPlay() {
	//已经开始播放
	if (_status.isPlaying == true) {
		return false;
	}
	//首次点击
	if (_status.firstClick == false) {
		_req = new URLRequest("test.mp3");
		_sound.load(_req);
		_status.firstClick = true;
	}
	//开始播放
	_channel = _sound.play(_status.pausePosition,1);

	//初始化暂停位置
	_status.pausePosition = 0;
	_status.isPlaying = true;

	//增加完成监听
	_channel.addEventListener(Event.SOUND_COMPLETE, onPlaybackComplete);
}
```

至于播放完毕后的监听，是为了将状态信息清零以便不影响到以后的播放。

```js
function onPlaybackComplete(event:Event) {
	_status.isPlaying = false;
}
```

#### 停止与暂停

对于暂停操作，首先要明白一点：AS3中并没有提供"暂停"声音播放这样的功能，要实现暂停，只能每次记录播放位置，将声音停止，然后再从停止的位置开始播放。

清楚了这点，那么可以知道停止和暂停操作唯一的区别就是是否记录播放位置，所以这里将播放位置作为入口参数构造出停止操作：

```js
function actionStop(position = 0) {
	if (_status.firstClick == true) {
		//停止播放
		_channel.stop();
		//记录停止位置
		_status.pausePosition = position;
		_status.isPlaying = false;
	} else {
		trace("Nothing to stop");
	}
}
```

然后在有了停止和播放两个操作的基础上实现暂停功能：

```js
function actionPause() {
	if (_status.pausePosition == 0 && _status.isPlaying == true) {
		actionStop(_channel.position);
	} else if (_status.pausePosition > 0 && _status.isPlaying == false) {
		actionPlay();
	} else {
		trace("Nothing to Pause");
	}
}
```

#### 整合

将写好的动作嵌入绑定的事件里，一个具有播放，停止，暂停功能的播放器便初步完成了。

```js
function onPlay(event:MouseEvent):void {
	actionPlay();
}
function onPause(event:MouseEvent):void {
	actionPause();
}
function onStop(event:MouseEvent):void {
	actionStop(0);
}
```

当然不要忘记引用所有用到的类

```js
import flash.events.*;
import flash.media.*;
import flash.display.*;
import flash.net.URLRequest;
```

最后仍然是本次[代码下载](http://cid-01e48df64f8bd957.skydrive.live.com/embedrowdetail.aspx/Source/avplayer2.7z)

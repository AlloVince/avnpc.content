---
title: "PHP上传问题总结(文件大小检测,大文件上传)"
date: "2008-10-01 15:22:18"
slug: "check_filesize_before_upload"
published: true
author: "AlloVince"
legacy_id: 114
comment_status: "closed"
comments: false
tags:
  - "FileReference"
  - "filesize"
  - "Flash"
  - "php"
  - "YD的程序员葛阁"
  - "上传"
---
**目录** [折叠]

1. [通过MAX_FILE_SIZE](#toc_08965300)
2. [通过服务器端](#toc_08975900)
3. [通过Javascript](#toc_08999400)
4. [通过Flash](#toc_09008200)
5. [结论](#toc_09034400)

由于涉及到本地和服务器两方面的安全问题，所以基于input type="file"形式的页面文件上传一直处于一个很尴尬的位置。一方面，用户不希望隐私泄露，所以浏览器无法对用户在上传时选择的文件做有效的判断。另一方面，为了服务器端的安全，减轻传输负担，系统又希望能在用户开始上传之前就将非法的文件拒之门外。

一来一去，基于原始input方式的上传，成为网络存储网站避之唯恐不及的遗留性问题，也造就了现在千奇百怪的插件、上传客户端。

input方式的上传就如此之差么？当然不是。上传文件不大的时候，它还是非常简单可靠的，在PHP中，我们只需要一个复合型表单

```xml
<form enctype="multipart/form-data" action="__URL__" method="POST">
```

一个输入框

```xml
<input name="userfile" type="file" />
```

和服务器端的一行代码

```php
move_uploaded_file($_FILES['userfile']['tmp_name'], '/var/www/uploads/'. basename($_FILES['userfile']['name']));
```

就可以实现整个上传过程。

但随文件增大，表单上传的不足就会暴露出来。尤其是我们想取得最基本的文件大小来阻止过大文件上传这一简单的想法，也变得如此困难。以下一一道来：

#### 通过MAX_FILE_SIZE

我们经常会在手册里读到:

> MAX_FILE_SIZE 隐藏字段（单位为字节）必须放在文件输入字段之前，其值为接收文件的最大尺寸。这是对浏览器的一个建议，PHP 也会检查此项。在浏览器端可以简单绕过此设置，因此不要指望用此特性来阻挡大文件。实际上，PHP 设置中的上传文件最大值是不会失效的。但是最好还是在表单中加上此项目，因为它可以避免用户在花时间等待上传大文件之后才发现文件过大上传失败的麻烦。

显然PHP的开发者们也考虑到了大文件上传的问题，但就像手册所说，MAX_FILE_SIZE只是对浏览器的一个建议，事实上目前为止所有主流的浏览器并没有采纳这个建议，所以采用MAX_FILE_SIZE约束文件大小形同摆设，不可行。

#### 通过服务器端

MAX_FILE_SIZE既然无效，那么用户可以将文件上传到服务器，服务器端通过$\_FILES['userfile']['size']判断用户上传的文件大小，然后决定是否接受上传并返回信息。暂且排除服务器的负荷以及可能存在的恶意破坏行为，这种解决方案听起来无非是浪费一部分带宽，也能对用户上传文件作出约束。

但这也是不可行的，PHP的文件上传受到php.ini以下这些设置的影响:

- post_max_size
- upload_max_filesize
- max_execution_time
- memory_limit

虽然设置方法在[手册](http://www.php.net/manual/en/features.file-upload.common-pitfalls.php)中都有比较详细的说明，之所以仍然说此方法不可行，是因为php执行脚本在超过memory_limit时，该次的POST数据会全部丢失并且不会报错！

试想用户填写了一个超长的表单，并伴随一个超过memory_limit的文件一起上传，经过了漫长的等待时间之后发现等来的又是一张干干净净的空白表单，那是何等印象深刻的用户体验啊。更何况数十M的服务器流量仅仅用来检测文件大小，是现在的网络环境不允许的。

#### 通过Javascript

Javascript是基于浏览器的，虽然JS能完成很多看似不可能的任务，但浏览器做不到的事情JS同样无法做到。先天不足注定了这项工作仅仅靠Javascript是无法胜任的。不过一些[IE Only的方法](http://www.ugia.cn/?p=73)也还是存在的，仅作[参考](http://hspinfo.wordpress.com/2008/05/30/using-javascript-check-uploaded-file-size-which-is-uploaded-by-a-file-upload-control/)。

#### 通过Flash

Flash的FileReference类提供了一套比较全面的文件处理方法，现在大多数大文件上传也都采用了基于Flash的方案。如果利用Flash与Js交互，能否实现客户端对文件大小的检测呢？答案是可行的。

首先在flash文件中实例化FileReference类。

```js
var fr = new FileReference();
```

基于这个类就可以用Flash提供的file browse和SelectFile事件替代浏览器的事件。我们需要：

1.绑定SelectFile

```js
fr.addEventListener(Event.SELECT, onSelectFile);
```

2.创建一个供Js访问的对象，用来放置flash得到的文件信息

```js
var s = {
	size:0,
	name:'',
	type:''
}
```

3.创建file browse方法

```js
function browseFile():void {
	fr.browse();
}
```

4.当SelectFile事件触发的时候，传递文件信息

```js
function onSelectFile(e:Event):void {
	s.size = fr.size;
	s.name = fr.name;
	s.type = fr.type;
}
```

5.将browseFile方法公开可供Js调用

```js
ExternalInterface.addCallback("browseFile", browseFile);
```

6.将得到的文件信息传递给Js

```js
ExternalInterface.call("onSelectFile",s);
```

现在我们已经可以通过Js获得由flash传递来的文件大小信息了，具体的实现可以参看[Demo](lab/upload_check/uploadCheck.html)。

#### 结论

问题至此似乎已经得到解决了，我们已经成功的校验了文件大小不是么。但本文的最终结论是，基于Flash的文件大小校验，仍然不可行。

文件大小校验的唯一目的，是为了上传。在上面的Demo中可以看到校验成功的文件名会显示在一个输入框里。熟悉上传的同学不觉得少了什么吗？没错，通过flash只能得到文件名，而无法得到文件的完整路径，而文件路径却是input方式上传的必要条件。所以虽然可以成功的通过Flash与Js交互校验文件大小，但我们能做到的也仅仅只是校验而已，之后想要上传，唯有继续通过flash方式进行。

Flash开发出于安全考虑屏蔽了文件的完整路径这无可厚非，不过文件上传，尤其是PHP环境下的文件校验上传方案仍然没有得到最好的解决。

当然弥补的方法有很多：

- 基于Perl的项目 [FileChucker](http://encodable.com/filechucker/) , [XUpload](http://www.sibsoft.net/xupload.html) , [Uber-Uploader](http://uber-uploader.sourceforge.net/)
- 基于Flash的项目 [SWFUpload](http://www.swfupload.org/)
- 还有筒子用PHP直接[在服务器华丽的建立socket链接](http://www.ugia.cn/?p=70)。

但终究我希望有一天能看到仅基于HTML就能实现的严整健壮的上传方案，但愿这一天不会太远。

最后是本次的[代码下载](http://cid-01e48df64f8bd957.skydrive.live.com/embedrowdetail.aspx/Source/uploadcheck.7z)。

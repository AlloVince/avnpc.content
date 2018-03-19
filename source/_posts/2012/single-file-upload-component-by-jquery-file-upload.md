---
slug: single-file-upload-component-by-jquery-file-upload
published: true
date: '2012-10-19 23:04:34'
tags:
  - jQuery
  - 上传
  - jQuery File Upload
  - 插件
author: AlloVince
title: 定制jQuery File Upload为微博式单文件上传
---

[jQuery File Upload](https://github.com/blueimp/jQuery-File-Upload)是一个非常优秀的上传组件，主要使用了[XHR](http://en.wikipedia.org/wiki/XMLHttpRequest)作为上传方式，并且利用了相当多的现代浏览器功能，所以可以实现诸如批量上传、超大文件上传、图片预览、拖拽上传、上传进度显示、跨域上传等功能。

美中不足的是[jQuery File Upload的默认UI](http://blueimp.github.com/jQuery-File-Upload/)比较复杂，集成了全部功能，让jQuery File Upload的定制变得比较繁琐。


尝试[用jQuery File Upload制作了一个类似微博图片上传的单文件式上传](http://avnpc.com/pages/single-file-upload-component-by-jquery-file-upload)Demo，将一些要点记录下来备忘。最终效果如下图：

![jQuery File Upload Demo](https://lh4.googleusercontent.com/-0gdZVs-1O8E/UIEZJ6grlTI/AAAAAAAAEzQ/-649GHgw6As/s288/jQuery%2520File%2520Upload%2520demo.jpg)

jQuery File Upload的最简模型
---------

jQuery File Upload包含了一堆文件，首先需要弄清楚的是最核心的部分是哪些，根据[官方的例子](https://github.com/blueimp/jQuery-File-Upload/wiki/Basic-plugin)可以知道，一个最简单的jQuery File Upload上传组件，必须包括以下文件：

- jQuery核心库，建议使用jQuery 1.8以上版本
- js/vendor/jquery.ui.widget.js ： [jQuery UI Widget](http://api.jqueryui.com/jQuery.widget/) 
- js/jquery.iframe-transport.js ： 扩展iframe数据传输
- js/jquery.fileupload.js ： jQuery File Upload核心类
- js/cors/jquery.xdr-transport.js 在IE下应载入此文件解决跨域问题

此时只需要加载一个上传按钮

    <input id="fileupload" type="file" name="files[]" data-url="server/php/" multiple>

以及一行代码

    $('#fileupload').fileupload();

就完成了一个最基本的上传组件。这个最简单的上传组件可以将选中的文件以表单形式提交到data-url约定的URL，同时提供了足够多的[设置和基础事件](https://github.com/blueimp/jQuery-File-Upload/wiki/Options)可供扩展。

jQuery File Upload的简单扩展
============

对于最简模型，稍加扩展就可以实现一些比较常用的功能，比如可以在上传完毕后可以显示一个简单的结果：

    $('#fileupload').fileupload({
        done: function (e, data) {
            $.each(data.result, function (index, file) {
                $('<p/>').text(file.name + ' uploaded').appendTo($("body"));
            });
        }
    });

或者显示上传进度，配合一些进度条组件就可以构成一个上传进度条

    $('#fileupload').fileupload('option', {
        progressall: function (e, data) {
            var progress = parseInt(data.loaded / data.total * 100, 10);
            console.log(progress + '%');
        }
    });
    
等等。只要多阅读[手册](https://github.com/blueimp/jQuery-File-Upload/wiki/Options)就可以配合项目做更具体的扩展开发。

XHR响应为Json时IE的下载BUG
------------

这里需要特别注意的是，由于jQuery File Upload都是采用XHR在传递数据，服务器端返回的通常是JSON格式的响应，但是IE会将这些JSON响应误认为是文件传输，然后直接弹出下载框询问是否需要下载。

解决这个问题的方法是必须将相应的Http Head从

    Content-Type: application/json

更改为

    Content-Type: text/plain

具体的实现根据服务端不同有所区别，比如[ZF2](http://avnpc.com/pages/zf2-summary)中可以在Controller中这样写：

     $this->getServiceLocator()->get('Application')->getEventManager()->attach(\Zend\Mvc\MvcEvent::EVENT_RENDER, function($event){
	     $event->getResponse()->getHeaders()->addHeaderLine('Content-Type', 'text/plain');
	 }, -10000);

这也是我在stackoverflow上的对[ZF2更改最终响应类型的一个回答](http://stackoverflow.com/questions/12820415/how-to-tell-zf2s-jsonmodel-to-return-text-plain-instead-of-application-json)


jQuery File Upload UI的构成与说明
============

为了引入更多功能，jQuery File Upload在上面最简模型的基础上又实现了一套jQuery File Upload UI，也就是官方给出的最终Demo，这套UI额外提供了以下功能：


- 最大/最小文件限定 Options.maxFileSize / Options.mixFileSize
- 文件类型限定，通过正则表达式检测文件名实现 Options.acceptFileTypes
- 选择文件后自动上传 Options.autoUpload
- 上传文件数量限制，通过上传后将选择文件按钮置为Disabled实现 Options.maxNumberOfFiles
- 上传模板，就是选择文件后显示预览的html代码 Options.uploadTemplate
- 下载模板，当文件上传完毕后显示的html代码 Options.downloadTemplate

等等，同时还增加了一系列新的接口和事件，具体都可以查阅官方手册。

具体对应到文件为：

- [JavaScript-Templates](http://blueimp.github.com/JavaScript-Templates/tmpl.min.js) : JS模板引擎
- [JavaScript-Load-Image](http://blueimp.github.com/JavaScript-Load-Image/load-image.min.js) : 图片预览功能
- js/jquery.fileupload-ui.js & css/jquery.fileupload-ui.css ： UI核心类，CSS可以替换旧式的上传控件为统一的按钮
- js/jquery.fileupload-fp.js：进度条扩展功能


也许正是因为附加功能太多，各功能之间耦合非常重，jQuery File Upload UI显得不够友好，主要体现在：

- 上述功能均无法拆分，必须统一全部加载
- 各功能需要界面存在相应元素，如果缺少某些元素，包括JS模板内的元素，整个UI无法正常工作
- JS模板引擎对标签配对非常严格，标签如果遗漏也有可能引起UI无法正常工作

所以经验之谈是，在定制jQuery File Upload UI时，如果UI无法工作。首先检查js文件是否全部加载，然后检查页面元素是否齐全，再次检查JS模板标签是否严格配对，最后还可以查看页面是否有重复调用fileupload()方法。

jQuery File Upload UI构成元素
-----------------

UI的部件都是硬编码的HTML class，无法更改。核心的几个部件为

###全局控制按钮 (必须)

        <div class="fileupload-buttonbar">
                <span class="fileinput-button"><input type="file" name="files[]" multiple></span>
                <button type="submit" class="start">Start upload</button>
                <button type="reset" class="cancel">Cancel upload</button>
                <button type="button" class="delete">Delete</button>
                <input type="checkbox" class="toggle">
        </div>


最外层容器为.fileupload-buttonbar，内部包含

 - 文件选择按钮 .fileinput-button (必须)，内部必须包裹一个input:file
 - 开始上传按钮 .start
 - 取消上传按钮 .cancel
 - 删除按钮 .delete
 - 文件勾选按钮 .toggle

###整体上传进度 (可选)

    <div class="fileupload-progress">
        <div class="progress">
            <div class="bar" style="width:0%;"></div>
        </div>
        <div class="progress-extended"></div>
    </div>

最外层容器为.fileupload-progress，内部包含

 - 上传进度条容器.progress
 - 上传进度条 .bar
 - 上传进度文本 .progress-extended

###文件显示容器 (必须)

    <div class="files"></div>

.file容器是最重要的UI部件，上传时的文件预览模板以及上传完毕后的文件显示模板都将显示在这里。

###文件预览模板 (必须)

    <script id="template-upload" type="text/x-tmpl">
	{% for (var i=0, file; file=o.files[i]; i++) { %}
	<div class="template-upload">
	    {% if (file.error) { %}
	        <div class="error">{%=file.error%}</div>
	    {% } else { %}
	    <div class="preview"><span class="fade"></span></div>
	    <div class="name"><span>{%=file.name%}</span></div>
	    <div class="size"><span>{%=o.formatFileSize(file.size)%}</span></div>
	    <div class="progress progress-success progress-striped active" role="progressbar" aria-valuemin="0" aria-valuemax="100" aria-valuenow="0" style="height:5px;"><div class="bar" style="width:0%;"></div></div>
	    <span class="start">
	        {% if (!o.options.autoUpload) { %}
	            <button>Start Upload</button>
	        {% } %}
	    </span>
	    {% } %}
	    <span class="cancel"><button>Cancel</button></span>
	</div>
	{% } %}
	</script>

这部分逻辑不难读懂，由于文件选择是多选的，所以被选择文件一开始以数组方式存放，循环输出。即使我们加入最大文件只能上传一个，这里得到的仍然是数组形式。

当文件有任何错误时，如文件类型被禁止，文件大小不符合约定，会得到file.error。文件检测没有问题，则可以用以下元素控制当前文件：

 - 开始上传当前文件按钮.start (必须)
 - 取消上传当前文件按钮.cancel (可选)
 - 当前文件上传进度.progress (可选)


###上传后文件回调显示模板 (必须)

    <script id="template-download" type="text/x-tmpl">
	{% for (var i=0, file; file=o.files[i]; i++) { %}
	<div class="template-download">
	    {% if (file.error) { %}
	        <div class="error">{%=file.error%}</div>
	        <span class="cancel"><button class="btn btn-block"><i class="icon-ban-circle"></i>Cancel</span>
	    {% } else { %}
	    <div class="preview"><img src="{%=file.thumbnail_url%}"></div>
	    <div class="name"><span>{%=file.name%}</span></div>
	    <div class="size"><span>{%=o.formatFileSize(file.size)%}</span></div>
	    <div class="delete"><button data-type="{%=file.delete_type%}" data-url="{%=file.delete_url%}">Delete</button>
	    </div>
	    {% } %}
	</div>
	{% } %}
	</script>

这一部分的o.files完全来自服务器端的json响应，所以模板内容可以自由发挥。唯一被定制的元素为删除按钮.delete。 点击这个按钮会向按钮中指定的url发送请求，比如

    <div class="delete"><button data-type="DELETE" data-url="/file/1">Delete</button></div>

点击后则会用DELETE方式发送HTTP请求

    DELETE /file/1

jQuery File Upload UI工作流程
-------------------

有了上面罗列的UI元素，就可以拼凑出一个简单的jQuery File Upload UI工作流程：

1. 用户点击.fileinput-button选择要上传的文件（多个）
2. 文件选择后，文件信息被整理为数组置入文件预览模板#template-upload
3. 模板引擎循环处理文件信息并生成模板.template-upload
4. 每生成一个模板，模板就被插入到文件显示容器.files的最后。
5. 用户点击上传按钮.start上传，文件信息被转换为XHR请求至服务器端
6. UI获得服务器端生成JSON响应文件
7. JSON响应信息也被整理成数组置入回调显示模板#template-download
8. 模板引擎循环处理文件信息并生成模板.template-download
9. 每生成一个模板，会将此模板替换对应的.template-upload部分


定制过程
============

有了上面的基础，要个性化的定制jQuery File Upload就简单了很多：

限制文件类型
-------------

由于没有使用Flash空间，上传的文件选择框是无法限制文件类型的，所以所谓的限制文件类型，只能让用户选择文件之后，用file.error显示一个错误信息。例如本次需要限定可上传的文件为图片，那么Options指定：

    acceptFileTypes:  /(\.|\/)(gif|jpe?g|png)$/i

即可。

在Google Chrome浏览器中，可以用input:file原生支持文件类型限定，可以配合使用：

    <input type="file" name="upload[]"  accept="image/png, image/gif, image/jpg, image/jpeg">

不过在客户端做再多的限定也只是提升用户体验，不能真正保证安全性，所以不要忘记了在服务器端做同样的类型检测。

文件数量限制
---------------

只需在Options指定

    maxNumberOfFiles : 1

即可。jQuery File Upload UI的处理方式是当用户上传一个文件后，文件选择按钮被置为Disabled。

这同样只是客户端的小把戏，真正想要严格的约束用户只能上传一个文件还是需要在服务器端通过Session做更加复杂的控制。

文件大小限制
---------------

Options中指定

    maxFileSize: 5000000

即只允许单文件最大5MB。


Firefox disable bug
-------------------

在Firefox环境下测试是，发现如果将文件数量限制为1，选择一次文件，刷新页面之后文件选择按钮会莫名其妙的被加上一个Disabled属性，导致无法点击。所以最终我们的初始化代码为：


    var uploader = $("#fileupload");
    uploader.fileupload({
        dataType: 'json',
        autoUpload: false,
        acceptFileTypes:  /(\.|\/)(gif|jpe?g|png)$/i,
        maxNumberOfFiles : 1,
        maxFileSize: 5000000 
    });
    uploader.find("input:file").removeAttr('disabled');


最后就是界面的一些调整，完整代码在[EvaEngine](http://avnpc.com/pages/eva-engine)的File模块下，[点击查看](https://github.com/AlloVince/eva-engine/blob/master/module/File/view/file/index.phtml).


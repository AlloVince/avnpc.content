---
slug: evabootstrap
published: true
date: '2012-11-12 22:29:26'
tags:
  - EvaEngine
  - Bootstrap
  - Project
author: AlloVince
title: EvaBootstrap - 无侵入的TwitterBootstrap定制版
---

[EvaBootstrap](https://github.com/AlloVince/EvaBootstrap)是一个基于[Twitter Bootstrap](http://twitter.github.com/bootstrap/)（下简称TB）的修改定制版，主要解决了TB的侵入性问题，能使TB更好的与其他前端项目并存。简言之，EvaBootstrap提供TB的所有特性，并且不会修改你的CSS默认样式。

EvaBootstrap将作为[EvaEngine](https://github.com/AlloVince/eva-engine)项目的前端核心框架开源发布，协议采用[New BSD License](http://framework.zend.com/license/new-bsd)。

目前EvaBootstrap基于Twitter Bootstrap v 2.2.1修改。


与Twitter Bootstrap的不同
--------------------------------

技术上来讲，EvaBootstrap的实现原理就是在TB有侵入性的代码上加入了一些CSS Class命名空间，所以使用时会与TB有细微的差别：

###文本排版

对于需要文本排版的元素，需要增加一个class .typo:


在Twitter Bootstrap中:

    <p><small>This line of text is meant to be treated as fine print.</small></p>

在EvaBootstrap中:

    <div class="typo">
    <p><small>This line of text is meant to be treated as fine print.</small></p>
    </div>

###表单

对于希望采用TB样式的表单，增加一个class .form

在Twitter Bootstrap中:

    <form>
    <fieldset>
    ...
    </fieldset>
    </form>

在EvaBootstrap中:

    <form class="form">
    <fieldset>
    ...
    </fieldset>
    </form>


###移除TB font-face图标

TB的图标略显简陋，索性完全移除。如果想要使用图标，推荐更全更精致的[Font Awesome](http://fortawesome.github.com/Font-Awesome/)，可以对应绝大多数项目需求。


新功能
-----------

###中文排版

通过引入[typo.css](https://github.com/sofish/typo.css)项目，提供更好的中文排版支持。

对于需要采用中文排版的元素，加上class .typocn

###高级别CSS Reset

TB的CSS Reset采用的是[normalize.css](http://necolas.github.com/normalize.css/1.0.1/normalize.css)，这种Reset以修复浏览器缺陷为主要目的，在实际项目中往往统一元素的初始状态，所以CSS Reset部分采用了更“高”的Reset，包括了元素样式级别的处理。

整个Reset部分可以选择加载，所以如果你的项目中已经有了Reset，只需引入evabootstrap-core即可。


###新的LESS文件结构与函数

将TB的less源文件按功能分类存放，方便查找。同时加入了一些TB中没有包含的便捷LESS函数。

###百分比Grid布局

作为TB Grid布局的一个补充，引入了一个非常小巧的[百分比Grid布局](http://cssglobe.com/easy-percentage-grid-system-with-html5/)

###三栏式布局

如何使用
-----------

在页面中根据需求引入对应的CSS即可

- evabootstrap-core.css EvaBootstrap的核心CSS，不包含Reset，不包含Responsive Layout，无任何侵入性
- evabootstrap-fixed.css 在evabootstrap-core的基础上增加了Reset
- evabootstrap-full.css 在evabootstrap-fixed.css的基础上增加了Responsive Layout

更多情况可以直接采用LESS编写，按需求引入LESS源文件即可。

意见反馈
--------------

请在[Blog](http://avnpc.com/pages/evabootstrap)留言或在[项目页](https://github.com/AlloVince/EvaBootstrap)提交BUG




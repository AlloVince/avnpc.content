---
published: true
date: '2012-11-12 22:29:26'
tags:
  - EvaEngine
  - Bootstrap
  - Project
author: AlloVince
title: EvaBootstrap - 无侵入的 TwitterBootstrap 定制版
---

[EvaBootstrap](https://github.com/AlloVince/EvaBootstrap)是一个基于[Twitter Bootstrap](http://twitter.github.com/bootstrap/)（下简称 TB）的修改定制版，主要解决了 TB 的侵入性问题，能使 TB 更好的与其他前端项目并存。简言之，EvaBootstrap 提供 TB 的所有特性，并且不会修改你的 CSS 默认样式。

EvaBootstrap 将作为[EvaEngine](https://github.com/AlloVince/eva-engine)项目的前端核心框架开源发布，协议采用[New BSD License](http://framework.zend.com/license/new-bsd)。

目前 EvaBootstrap 基于 Twitter Bootstrap v 2.2.1 修改。


与 Twitter Bootstrap 的不同
--------------------------------

技术上来讲，EvaBootstrap 的实现原理就是在 TB 有侵入性的代码上加入了一些 CSS Class 命名空间，所以使用时会与 TB 有细微的差别：

###文本排版

对于需要文本排版的元素，需要增加一个 class .typo:


在 Twitter Bootstrap 中:

```html
<p><small>This line of text is meant to be treated as fine print.</small></p>
```

在 EvaBootstrap 中:

```html
<div class="typo">
<p><small>This line of text is meant to be treated as fine print.</small></p>
</div>
```

###表单

对于希望采用 TB 样式的表单，增加一个 class .form

在 Twitter Bootstrap 中:

```html
<form>
<fieldset>
...
</fieldset>
</form>
```

在 EvaBootstrap 中:

```html
<form class="form">
<fieldset>
...
</fieldset>
</form>
```


###移除 TB font-face 图标

TB 的图标略显简陋，索性完全移除。如果想要使用图标，推荐更全更精致的[Font Awesome](http://fortawesome.github.com/Font-Awesome/)，可以对应绝大多数项目需求。


新功能
-----------

###中文排版

通过引入[typo.css](https://github.com/sofish/typo.css)项目，提供更好的中文排版支持。

对于需要采用中文排版的元素，加上 class .typocn

###高级别 CSS Reset

TB 的 CSS Reset 采用的是[normalize.css](http://necolas.github.com/normalize.css/1.0.1/normalize.css)，这种 Reset 以修复浏览器缺陷为主要目的，在实际项目中往往统一元素的初始状态，所以 CSS Reset 部分采用了更“高”的 Reset，包括了元素样式级别的处理。

整个 Reset 部分可以选择加载，所以如果你的项目中已经有了 Reset，只需引入 evabootstrap-core 即可。


###新的 LESS 文件结构与函数

将 TB 的 less 源文件按功能分类存放，方便查找。同时加入了一些 TB 中没有包含的便捷 LESS 函数。

###百分比 Grid 布局

作为 TB Grid 布局的一个补充，引入了一个非常小巧的[百分比 Grid 布局](http://cssglobe.com/easy-percentage-grid-system-with-html5/)

###三栏式布局

如何使用
-----------

在页面中根据需求引入对应的 CSS 即可

- evabootstrap-core.css EvaBootstrap 的核心 CSS，不包含 Reset，不包含 Responsive Layout，无任何侵入性
- evabootstrap-fixed.css 在 evabootstrap-core 的基础上增加了 Reset
- evabootstrap-full.css 在 evabootstrap-fixed.css 的基础上增加了 Responsive Layout

更多情况可以直接采用 LESS 编写，按需求引入 LESS 源文件即可。

意见反馈
--------------

请在[Blog](http://avnpc.com/pages/evabootstrap)留言或在[项目页](https://github.com/AlloVince/EvaBootstrap)提交 BUG




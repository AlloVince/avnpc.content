---
title: "非中文系统下Gvim中文化解决方法"
date: "2008-05-07 16:29:55"
slug: "Gvim_encoding"
published: true
author: "AlloVince"
legacy_id: 92
comment_status: "closed"
comments: false
tags:
  - "Vim"
  - "YD的程序员葛阁"
  - "乱码"
---
本日志已经更新，请移步：[http://avnpc.com/pages/vim-of-allovince](/pages/vim-of-allovince)

Gvim在安装时会根据操作系统的编码自动选择相应的语言包，但有时候想要强制选择自己指定的语言时就需要进行配置。下面以日文系统下Gvim的中文化为例。

- set encoding=utf-8
- 设定Gvim的内部文字编码为utf-8
- set langmenu=zh_CN.UTF-8
- 设定Gvim的菜单使用中文表示
- language message zh_CN.UTF-8
- 这里将Gvim的指令提示、帮助文档等设定为中文
- set guifont=NSimSun:h10
- 其实至第三步为止，Gvim已经可以显示中文了，但由于安装在非中文系统下，Gvim会选择系统默认字体作为自己的GUI字体，所以一般这里仍不能正常显示，所以还需要设置一个包含中文字库的字体
- set fileencodings=ucs-bom,utf-8,chinese,japanese
- 这里是设置文件打开的解码顺序，从最严格的ucs-bom开始尝试解码，不成功则转向下一个。

另外为了更好的让Gvim自动识别文件内码，可以使用[FencView插件](http://www.vim.org/scripts/script.php?script_id=1708)

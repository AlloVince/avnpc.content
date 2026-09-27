---
title: "Vim配置外一篇(中文化、输入法、代码补全等)"
date: "2008-07-08 14:24:51"
slug: "Vim_Config"
published: true
author: "AlloVince"
legacy_id: 102
comment_status: "closed"
comments: false
tags:
  - "Vim"
  - "YD的程序员葛阁"
---
**目录** \[[折叠](../posts/javascript:;)]

1. [多语言平台下的中文化实现](#toc_66127000)
2. [IME输入法切换设置](#toc_66139800)
3. [代码自动补全](#toc_66150100)
4. [自定义快捷键](#toc_66161600)
5. [至懒人](#toc_66175500)
6. [至更懒的人](#toc_66189800)

本日志已经更新，请移步：[http://avnpc.com/pages/vim-of-allovince](../../pages/vim-of-allovince)

Vim不愧是地球上最强的文本编辑器之一（不加之一恐怕Emacs用户会有意见- -），用Vim就像用Foobar一样，必须先调教一番，使之很好的符合自己的习惯才能达到最高的效率。

之前（前情提要：[非中文系统下Gvim中文化解决方法](../posts/blog/view/91)）提到了Vim在其他语言Win32平台下的中文化实现，这里再做一些补充修订：

#### 多语言平台下的中文化实现

首先是编码设置

```sql
"设置内核为UTF-8
set enc=utf-8
"设置解码顺序,这个顺序经测试可以很好的识别UTF-8,JIS,GBK
set fencs=utf-8,ucs-bom,shift-jis,gb18030,gbk,gb2312,cp936
```

然后设置界面

```sql
"语言设置
set langmenu=zh_CN.UTF-8
language message zh_CN.UTF-8
set guifont=NSimSun:h10
set helplang=cn
source $VIMRUNTIME/delmenu.vim
source $VIMRUNTIME/menu.vim
```

注意这里需要下载Vim的[中文文档](http://vcd.gro.clinux.org)

#### IME输入法切换设置

在Win32环境下，Vim总是会自作聪明的自动切换到IME输入法。如果想要关闭自动切换，只需要设置

```sql
set iminsert=0 imsearch=0
```

即可，但由于Vim提供了判断IME输入是否打开的multi_byte_ime，我们就可以更加智能的加一些处理

```sql
if has('multi_byte_ime')
	"未开启IME时光标背景色
	hi Cursor guifg=bg guibg=Orange gui=NONE
	"开启IME时光标背景色
	hi CursorIM guifg=NONE guibg=Skyblue gui=NONE
	" 关闭Vim的自动切换IME输入法(插入模式和检索模式)
	set iminsert=0 imsearch=0
	" 插入模式输入法状态未被记录时，默认关闭IME
	"inoremap <silent> <ESC> <ESC>:set iminsert=0<CR>
endif
```

以上设置可以记忆IME开启状态，并用不同的光标颜色提示当前输入法

#### 代码自动补全

实现代码补全需要插件[Ctags](http://ctags.sourceforge.net/),将下载的Ctags解压到任意文件夹，然后将Ctags目录追加到环境变量里，之后我们进入项目所在根目录，运行Ctags -R，Ctags会解析所有项目文件并生成一个类似索引的tags文件，然后Vim运行补全时会从tags文件中取得数据。

当然如果更新了类函数定义，也要再次运行Ctags重新生成一次索引，这没有Eclipse那样华丽而且智能，但相比较那些专用IDE的体积和启动速度，Vim的表现仍然非常出色。

默认的代码补全快捷键是Ctrl+X Ctrl+O，可以使用插件[SuperTab](http://www.vim.org/scripts/script.php?script_id=1643)将快捷键与Tab键合并

#### 自定义快捷键

Normal模式下可以很方便的用nmap设定自定义快捷键，不过可惜的是键盘上的键位也已经被占用的七七八八了。仅列举自己常用的两个作为例子：

```sql
nmap mm :%s/\r//g<cr>
```

双击m键可以删除Win下生成的多余换行符CR（在Vim中可以看到蓝色的^M）

```sql
nmap tt :%s/^\([\s　]\+\)/    /g<cr>:%s/^更新时间.*\d$//g<cr>:%s/<a href.*<\/a>$//g<cr>:%s/\([\s　]*\n\)\+/\r\r/<cr>
```

这个就比较复杂一点，双击t键实现对在起点下载的TXT文本进行排版并删除里面多余的广告等。

最终效果截图

![Vim of AlloVince](https://lh4.googleusercontent.com/-HG6Q7LnmsAA/SHR0wwKISNI/AAAAAAAAAb8/vEsc7McAEY0/s400/VIM.jpg)

#### 至懒人

好吧，贴出自己的配置，如果只想复制粘贴的同学可以各取所需，不过注意配置对应的插件

```sql
set nocompatible
source $VIMRUNTIME/vimrc_example.vim
source $VIMRUNTIME/mswin.vim
behave mswin

"设置工作目录为当前编辑文件的目录
set bsdir=buffer
set autochdir

"编码设置
set enc=utf-8
set fencs=utf-8,ucs-bom,shift-jis,gb18030,gbk,gb2312,cp936

"语言设置
set langmenu=zh_CN.UTF-8
language message zh_CN.UTF-8
set guifont=NSimSun:h10
set helplang=cn
source $VIMRUNTIME/delmenu.vim
source $VIMRUNTIME/menu.vim

"禁止生成临时文件
set nobackup

"搜索忽略大小写
set ignorecase

"搜索逐字符高亮
set incsearch

"行内替换
set gdefault

"始终显示行号
set nu!

"Tab键的宽度
set shiftwidth=4
set tabstop=4

"配色方案
colorscheme rainbow_fruit

"输入法设置
if has('multi_byte_ime')
	"未开启IME时光标背景色
	hi Cursor guifg=bg guibg=Orange gui=NONE
	"开启IME时光标背景色
	hi CursorIM guifg=NONE guibg=Skyblue gui=NONE
	" 关闭Vim的自动切换IME输入法(插入模式和检索模式)
	set iminsert=0 imsearch=0
	" 插入模式输入法状态未被记录时，默认关闭IME
	"inoremap <silent> <ESC> <ESC>:set iminsert=0<CR>
endif

"自动缩进
set ai!

"与Windows共享剪贴板
set clipboard+=unnamed

"Ctags的默认索引(需要插件Ctags)
set tags=$VIMRUNTIME/tags
"添加类型检测
filetype plugin indent on
"智能补全
"set completeopt=longest,menu

"快捷键调用WinManager(需要插件WinManager)
let g:winManagerWindowLayout='FileExplorer'
nmap ff :WMToggle<cr>
nmap mm :%s/\r//g<cr>
"规范行首空格<cr>去除多余字符<cr>删除空白行<cr>规范行数
nmap tt :%s/^\([\s　]\+\)/    /g<cr>:%s/^更新时间.*\d$//g<cr>:%s/<a href.*<\/a>$//g<cr>:%s/\([\s　]*\n\)\+/\r\r/<cr>

"通过ctrl+方向键切换窗口(需要插件MiniBufExplorer)
let g:miniBufExplMapCTabSwitchBufs = 1
let g:miniBufExplMapWindowNavArrows = 1
```

#### 至更懒的人

好吧，我放下载就是了…… [点击我](http://cid-01e48df64f8bd957.skydrive.live.com/embedrowdetail.aspx/Soft/Vim.7z)试试本Blog新增的下载功能吧，下载后解压到D:根目录，将Ctags目录（D:\Vim\vim71\plugin\ctags）加入环境变量即可

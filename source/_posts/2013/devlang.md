---
published: true
date: '2013-08-24 18:15:38'
tags: []
author: AlloVince
title: 《自制编程语言》相关资料
---

![自制编程语言](http://ec4.images-amazon.com/images/I/61JwHQaIoZL._SL500_AA300_.jpg)

[《自制编程语言》](http://www.ituring.com.cn/book/1159)原书名为[「プログラミング言語を作る」](http://www.amazon.co.jp/dp/4774138959/)，从编程语言的原理讲起，手把手地带你从零开始自制编程语言：crowbar 和 Diksam。前者为基于语法树的无类型语言，后者为基于字节代码的静态语言。二者均具备四则运算、变量、条件转移、循环、函数说明、垃圾收集、面向对象、异常处理机制等功能。 

《自制编程语言》原作者为[前橋和弥](http://kmaebashi.com/)，中文版由刘卓、[徐谦（AlloVince）](http://avnpc.com/)、吴雅明合译完成，[北京图灵文化发展有限公司](http://www.ituring.com.cn/)出版，现已出版。由于原版链接的资料多为作者的日语博客，考虑到本书读者大都没有日语基础，因此将[《自制编程语言》资源下载以及一些相关日志翻译](http://avnpc.com/pages/devlang)整理于此，希望可以帮助读者更方便的获取信息。

购买地址：

- [亚马逊](http://www.amazon.cn/dp/B00GAUNDYY/)
- [当当](http://product.dangdang.com/23363722.html)
- [京东](http://item.jd.com/11352448.html)
- [互动出版社](http://product.china-pub.com/3768792)


## 源代码下载 

作者提供的源代码中，错误信息全部为日文，给许多读者的学习带来了一些不便。

因此译者刘卓特别翻译了代码中所有的错误信息，并提供中文版代码的打包下载。

本书所涉及到的所有源代码：

- [中文版源代码 ZIP 压缩包，最后修订时间：20131212](https://static.avnpc.com/devlang/win_sjis_zh_20131212.zip)
- [日文版源代码 ZIP 压缩包，最后修订时间：20091228](https://static.avnpc.com/devlang/win_sjis_20091228.zip)


### 包含文件

压缩包内包含以下内容：

- calc　……第 2 章计算器源代码
    - mycalc　……基于 yacc/lex 的计算器源代码
    - mycalc_ex　……基于 yacc/lex 的计算器扩展版源代码
    - llparser　……基于递归下降分析法自制计算器源代码
    - llparser_ex　……基于递归下降分析法自制计算器扩展版源代码
- crowbar_book_0_1　……基于分析树运行的语言 crowbar ver.0.1。包含基本的语句、结构控制、函数
- crowbar_book_0_2　……基于分析树运行的语言 crowbar ver.0.2。引入数组与 GC
- crowbar_book_0_3　……基于分析树运行的语言 crowbar ver.0.3。内部编码采用宽字符（可支持日语）
- crowbar_book_0_4　……基于分析树运行的语言 crowbar ver.0.4。其他应用
- diksam_book_0_1　……基于静态字节码运行的语言 Diksam ver.0.1。包含基本的语句、结构控制、函数
- diksam_book_0_2　……基于静态字节码运行的语言 Diksam ver.0.2。引入数组
- diksam_book_0_3　……基于静态字节码运行的语言 Diksam ver.0.3。引入分割源代码及类
- diksam_book_0_4　……基于静态字节码运行的语言 Diksam ver.0.4。其他应用


### 编译方法


#### 计算器的编译

Linux 运行目录下的 make.sh，Windows 运行目录下的 make.bat 即可。


#### crowbar 的编译

在文件夹中运行 make（Windows 下可参考书中建议，重命名为 gmake）即可生成可执行文件。


#### Diksam 的编译

对于 diksam_book_0_1 及 diksam_book_0_2，解压后进入文件夹下的 compiler 目录，运行 make/gmake 即可。

对于 diksam_book_0_3 及 diksam_book_0_4，解压后进入文件夹下的 main 目录，运行 make/gmake 即可。


## MinGW, bison, flex 安装说明

下文介绍的所有安装文件均以 2013/8/23 的最新版本为准。


### MinGW 的安装

MinGW 的官方网站如下：

[http://www.mingw.org/](http://www.mingw.org/)

首先进入 MinGW 的[下载页面](http://www.mingw.org/download.shtml)，会跳转到[MinGW 在 sourceforge 的项目页](http://sourceforge.net/projects/mingw/files/)，Windows 用户可以点击 Installer，然后在列表中继续点击 mingw-get-setup.exe。

![MinGW](https://static.avnpc.com/devlang/devlang001.png)
![MinGW](https://static.avnpc.com/devlang/devlang002.png)

mingw-get-setup.exe 并不包含实际的安装文件，而是通过网络下载 MinGW 的程序文件。请参考下面步骤安装：

![MinGW](https://static.avnpc.com/devlang/devlang003.png)

点击`Install`。

![MinGW](https://static.avnpc.com/devlang/devlang004.png)

可在 Installation Directory 处点击 Change 修改安装路径，然后点击 Continue。

![MinGW](https://static.avnpc.com/devlang/devlang005.png)

MinGW 会下载初始安装文件，下载完成后点击 Continue。

![MinGW](https://static.avnpc.com/devlang/devlang006.png)

安装界面会切换为 MinGW Installation Manager，MinGW 包含了多个编译组件，选择想要安装的组件，在弹出的菜单中选择`Mark for Installation`。

《自制编程语言》一书中，我们只用到了`make`，因此只选择`mingw32-base`即可，当然如果磁盘空间富裕，完全可以全部选择，更加省事。

选择完毕后点击窗口菜单的 Installation，选择 Apply

![MinGW](https://static.avnpc.com/devlang/devlang007.png)

在弹出的确认框中点击 Apply，MinGW 会开始下载选择的安装包，整个下载过程视网速会占用数分钟的时间。下载完成即可以开始使用 MinGW。

安装完毕后可以在安装路径下看到 MinGW 创建了一系列文件，其中 bin 目录是 MinGW 主要程序所在目录。例如解压 crowbar 的源码到 c:，然后运行 cmd 如下编译 crowbar_book_0_1（假设 MinGW 的安装目录为`D:\MinGW`）：

```plain
cd C:\win_sjis\crowbar_book_0_1
C:\win_sjis\crowbar_book_0_1>D:\MinGW\bin\mingw32-make.exe
```

但是每次这样运行编译指令有点麻烦，所以作者推荐复制`mingw32-make.exe`到同目录下，并重命名为`gmake.exe`。然后将`D:\MinGW\bin`加入系统目录。就可以直接运行：

```plain
 gmake
```

进行编译了。


### Cygwin 的安装

Cygwin 是一系列自由软件的集合，可以在 Windows 下实现与 Linux 几乎一致的运行环境。

为了在 Windows 上实现 UNIX 的系统内核，Cygwin 提供了名为 cygwin.dll 的 DLL 文件来模拟 Linux 的内核 API 接口及相似功能。因此 Linux 源代码编译都依赖 cygwin.dll 文件，而通过 Cygwin 编译的 exe 文件如果脱离了包含 cygwin.dll 的系统环境也是无法运行的。

与 MinGW 一样，在 Cygwin 中也可以安装 gcc，而在《自制编程语言》一书中，作者之所以选择 MinGW 编译，主要是因为作者希望无论 crowbar 还是 Diksam，都可以最终编译为一个 exe 文件，只需要拷入 U 盘就能在任意电脑运行，不受 cygwin.dll 的限制。

而这里介绍 Cygwin 的安装，是由于 bison 所使用的 m4 以及 crowbar_ver.0.4 所使用的“鬼车”，都依赖于 Cygwin 环境。

Cygwin 的官方主页为：

[http://www.cygwin.com/](http://www.cygwin.com/)

进入主页后，可根据自己的系统环境选择下载 32bit 版本[setup-x86.exe](http://cygwin.com/setup-x86.exe)或 64bit 版本的[setup-x86_64.exe](http://cygwin.com/setup-x86_64.exe)。

下载后按以下方法操作（以 64bit 为例）

![Cygwin](https://static.avnpc.com/devlang/devlang008.png)

点击下一步


![Cygwin](https://static.avnpc.com/devlang/devlang009.png)

选择 Install from Internet，点击下一步


![Cygwin](https://static.avnpc.com/devlang/devlang010.png)

在 Root Directory 中选择安装路径，点击下一步


![Cygwin](https://static.avnpc.com/devlang/devlang011.png)

在 Local Package Directory 中选择安装包的下载路径，以后卸载或重新安装时可以从安装包直接获取，无需重新下载。点击下一步


![Cygwin](https://static.avnpc.com/devlang/devlang012.png)

一般选择 Direct Connection，如果网络环境特殊可根据具体情况设置代理。点击下一步

![Cygwin](https://static.avnpc.com/devlang/devlang013.png)

选择下载节点，国内一般选择 163 节点即可。Cygwin 会首先下载所有安装包信息。


![Cygwin](https://static.avnpc.com/devlang/devlang014.png)

选择要下载的模块安装包，Cygwin 已经为我们默认选择了一部分基础功能


![Cygwin](https://static.avnpc.com/devlang/devlang015.png)

这里我们为了使用 bison，需要在 Interpreters 下勾选 m4。而为了编译鬼车，需要勾选 Devel 下的 make。

Cygwin 会开始下载所有的安装包进行安装，同时在安装目录下生成 Linux 典型的目录结构。安装完毕后会在桌面生成 Cygwin64 Terminal 的快捷方式，双击之后就可以运行模拟的 Linux 环境了。


### bison 的安装

bison 最新版可以在下面的页面下载：

[http://gnuwin32.sourceforge.net/packages/bison.htm](http://gnuwin32.sourceforge.net/packages/bison.htm)

一般选择 Complete package, except sources 一栏的下载即可，下载后为 exe 格式安装文件，一路 next 安装后即可运行。


### flex 的安装

flex 最新版可以在下面的页面下载：

[http://gnuwin32.sourceforge.net/packages/flex.htm](http://gnuwin32.sourceforge.net/packages/flex.htm)

一般选择 Complete package, except sources 一栏的下载即可，下载后为 exe 格式安装文件，一路 next 安装后即可运行。


## 鬼车的安装 

鬼车是一个正则表达式程序库，官方主页为：

[http://www.geocities.jp/kosako3/oniguruma/](http://www.geocities.jp/kosako3/oniguruma/)

截至目前（2013/8/23）为止最新版本为 5.9.4。进入主页后点击[Latest release version 5.9.4](http://www.geocities.jp/kosako3/oniguruma/archive/onig-5.9.4.tar.gz)即可下载


### 鬼车在 Linux 下的安装

鬼车默认的系统环境为 Linux。在 Linux 下可以很简单的使用几行指令编译安装

```plain
wget http://www.geocities.jp/kosako3/oniguruma/archive/onig-5.9.4.tar.gz
tar -xvf onig-5.9.4.tar.gz
cd onig-5.9.4
./configure
make
make install
```


### 鬼车在 Windows 下的安装

对于 Windows 用户来说，我们需要借助 Cygwin 环境模拟 Linux 对其进行编译。

首先按照上文介绍的步骤安装好 Cygwin，由于鬼车要求 make 指令名必须为`make`，因此首先进入 MinGW 的 bin 目录，复制 mingw32-make.exe 并重命名为 make.exe。然后打开 cmd。假设 onig-5.9.4.tar.gz 文件下载到 C:\。运行以下指令进行解压并检查环境。

```plain
C:\>tar -xvf onig-5.9.4.tar.gz
C:\>cd onig-5.9.4
C:\onig-5.9.4>bash
bash-3.1$  ←命令提示符会切换为bash
./configure
make
```

理论上只要在 Linux 环境下，像鬼车这样基于[Autoconf](http://www.gnu.org/software/autoconf/)打包发布的软件都可以用与 Linux 相同的指令`make install`进行安装。

但是由于 Windows 下 MinGW 安装路径比较特殊，直接运行`make install`很可能报错。这里需要手动更新鬼车静态链接用库文件`ranlib.a`的索引。运行

```plain
bash-3.1$ cd .libs/
bash-3.1$ ranlib libonig.a
```

然后将 libonig.a 文件复制到 MinGW 安装目录下的 lib 文件夹内。最后运行

```plain
make install
```


## Hoge 一词的由来

在《自制编程语言》一书中，示例程序中经常使用`hoge`作为无意义字符串输出，而在英语程序则惯用`foobar`。

`hoge`写成日语假名是“ほげ”。那么究竟`hoge`是什么意思，为什么要用`hoge`作为无意义字符的的代表呢？作者对此进行了一系列[深入细致的考证与研究](http://kmaebashi.com/programmer/hoge.html)。由于其中涉及到很多日本的人名、地名、风俗习惯等等，这里只选择一些中国读者比较容易理解的部分翻译整理如下。


### Hoge 是什么意思

Hoge 是日本自古以来就口口相传的词汇，在为难、困惑、或者遇到麻烦事的时候都可以用 Hoge 来求救。据传 Hoge 一词最早的正式登场是在 20 世纪 80 年代前半，在日本各地突然同时开始流行。而使用“Hego”一词可供考证的最早记录，则是 1984 年曽田大明在名古屋大学所使用的某张 3.5 寸软盘中录入的 Hoge，说不定现在还能找到呢。

注：其实上面这些看似一本正经的说明大多都是作者在恶搞而已，Hoge 在日语中是一个非常口语化的词，根本无从考证。类比中文相当于要考证“我靠”一词的起源一样。而上文所说的“曽田大明”，应该是作者拿自己的某位同学在开涮吧。


### 什么时候使用 Hoge

- 在为文件、函数、变量命名而苦恼时可以使用 Hoge 来命名。
- 问别人一件事的时候，如果听不懂对方的回答，那么不要重复问题，简单说一个 Hoge 就可以了。
- 有时候面对电脑不知道输入些什么，那么不妨试试输入 Hoge 吧


### 谁在使用 Hoge

- 藤子不二雄的《哆啦 A 梦》：胖虎“动人”的歌唱声中必定会伴有 Hoge 的字眼
- 动画《多罗罗》的歌词中出现了 Hoge
- 新潟县的方言中一直有 Hogeru 的说法


## 《征服 C 指针》Web 版


请点击查看[《征服 C 指针》Web 版](http://avnpc.com/pages/c-pointer)

声明：

__原日文网页版“[配列とポインタの完全制覇](http://kmaebashi.com/programmer/pointer.html)”与出版后的「C 言語 ポインタ完全制覇」在内容上并不完全相同，这里对应截取中文简体版「C 言語 ポインタ完全制覇」（[《征服 C 指针》](http://www.ituring.com.cn/book/1036)）的内容。前桥和弥著，吴雅明译，人民邮件出版社，2013 年 2 月第 1 版。__


## 附录：《自制编程语言》日文版页面链接

- 作者主页： [K.Maebashi's home page](http://kmaebashi.com/)
- 自制编程语言专题页面： [「プログラミング言語を作る」書籍情報](http://kmaebashi.com/programmer/devlang/book/)
- 源代码下载页面： [「プログラミング言語を作る」ダウンロード](http://kmaebashi.com/programmer/devlang/book/download.html)
- 鬼车的安装：[鬼車のインストール](http://kmaebashi.com/programmer/devlang/book/oniguruma.html)
- Hoge 一词的由来：[ほげを考えるページ](http://kmaebashi.com/programmer/hoge.html)
-  《征服 C 指针》Web 版：[配列とポインタの完全制覇](http://kmaebashi.com/programmer/pointer.html)




---
published: true
date: '2012-12-05 22:53:17'
tags:
  - Python
  - Sphinx
  - Read the Docs
author: AlloVince
title: 写最好的文档：Sphinx + Read the Docs
---

写文档一向是个苦差事，但只有写出好的文档，才能有资格霸气十足的对别人淡淡道出:RTFD(Read the Fuck Document)。为了这一崇高目标，经过一些比较和调查，最终锁定[Sphinx + ReadTheDocs 作为文档写作工具](http://avnpc.com/pages/writing-best-documentation-by-sphinx-github-readthedocs)，以下逐一道来：

## 使用 Sphinx 生成文档

[Sphinx](http://sphinx-doc.org/)是一个基于 Python 的文档生成项目。最早只是用来生成 Python 的项目文档，但随着这个项目的逐渐完善，很多非 Python 的知名项目也采用 Sphinx 作为文档写作工具，甚至完全可以[用 Sphinx 来写书](http://hyry.dip.jp/tech/book/page/sphinx/index.html)。

引用一段[Sphinx 生成文档的优点](http://sphinx-doc-zh.readthedocs.org/en/latest/)包括：

 - 丰富的输出格式: 支持输出为 HTML，LaTeX (可转换为 PDF)， manual pages(man), 纯文本等若干种格式
 - 完备的交叉引用: 语义化的标签,并对 函式,类,引文,术语以及类似片段消息可以自动化链接
 - 明晰的分层结构: 轻松定义文档树,并自动化链接同级/父级/下级文章
 - 美观的自动索引: 可自动生成美观的模块索引
 - 精确的语法高亮: 基于 Pygments 自动生成语法高亮
 - 开放的扩展: 支持代码块的自动测试,自动包含 Python 的模块自述文档,等等

其实上面这么多功能，最本质的核心还是在于 Sphinx 采用了[轻量级标记语言](http://en.wikipedia.org/wiki/Lightweight_markup_language)中的[reStructuredText](http://docutils.sourceforge.net/rst.html)作为文档写作语言。reStructuredText 是类似 Wiki，Markdown 的一种纯文本标记语言，所有 Sphinx 的文档其实都是扩展名为 rst 的纯文本文件，然后经过转换器转换为各种输出格式，并且可以配合版本控制系统轻松实现 Diff。

### Sphinx 安装

首先安装好 Python 环境，建议选择 Pyhon2.7.3,并且把 Python 及 Python/Scripts 目录加入环境变量，然后只需要一行命令即可

```bash
easy_install -U Sphinx
```

安装完毕之后，进入任意目录，运行

```bash
sphinx-quickstart
```

会进入一个设置向导，根据向导一步一步设置文档项目，其实必填项只有项目名称，作者和版本，其他设置都可以一路回车：

 1. 文档根目录(Root path for the documentation)，默认为当前目录(.)
 2. 是否分离文档源代码与生成后的文档(Separate source and build directories): y
 3. 模板与静态文件存放目录前缀(Name prefix for templates and static dir):_
 4. 项目名称(Project name) : EvaEngine
 5. 作者名称(Author name)：AlloVince
 6. 项目版本(Project version) : 1.0.1
 7. 文档默认扩展名(Source file suffix) : .rst
 8. 默认首页文件名(Name of your master document):index
 9. 是否添加 epub 目录(Do you want to use the epub builder):n
 10. 启用 autodoc|doctest|intersphinx|todo|coverage|pngmath|ifconfig|viewcode：n
 11. 生成 Makefile (Create Makefile)：y
 12. 生成 windows 用命令行(Create Windows command file):y

最后会生成 Sphinx 一个文档项目必需的核心文件，包括：

```plain
    readthedocs
	│ make.bat
	│ Makefile
	├─build
	└─source
	　　│ conf.py
	　　│ index.rst
	　　├─_static
	　　└─_templates
```

如果向导中的所有设置都保存在 conf.py 中，可以随时调整。


### Sphinx 生成文档

source 目录就是存放文档源代码的目录，默认的索引页面为 index.rst

我们尝试来写作第一篇文档，在 source 目录下建立 helloworld.rst，内容为：

```plain
   Hello World
   ===========
```

同时编辑 index.rst 对应部分为

```plain
    .. toctree::
       :maxdepth: 1

       helloworld
```

然后在当前目录下运行

```plain
    make html
```

会看到 build 目录下会生成 HTML 格式的文档。同理我们可以 make letex 生成 LeTex 以及其他格式。


## 托管到 Read the Docs


[Read the Docs](http://readthedocs.org)是一个基于 Sphinx 的在线文档托管系统，接受一个 Git Repository 或 SVN 仓库作为文档来源。

我们将刚才的文档项目首先托管到 Github 上,因为 build 目录下的内容是自动生成的，所以还需要写一个.gitignore 将其忽略掉。

然后注册 Read the Docs，在 Dashboard 中创建一个新的 Project，Repo 中填入项目的 git url。

剩下的一切就交给 Read the Docs 吧。

我尝试创建的[EvaEngine Documentation](https://github.com/AlloVince/evaengine-documentation)，对应的[在线文档在此](https://evaengine.readthedocs.org/en/latest/)。

简单的流程就是这样，剩下的仍然是[RTFD](http://sphinx-doc.org/contents.html)。


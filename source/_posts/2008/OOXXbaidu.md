---
title: "OOXX baidu v0.1 release(百度空间转Wordpress搬家程序)"
date: "2008-04-09 01:44:15"
slug: "OOXXbaidu"
published: true
author: "AlloVince"
legacy_id: 86
comment_status: "closed"
comments: false
tags:
  - "baidu"
  - "php"
  - "Wordpress"
  - "YD的程序员葛阁"
  - "博客搬家"
  - "百度空间"
---
**目录** [折叠]

1. [功能](#toc_44803200)
2. [对应Wordpress版本](#toc_44809300)
3. [运行环境](#toc_44813600)
4. [使用方法](#toc_44817600)
5. [注意](#toc_44824200)

#### 功能

自动抓取hi.baidu（百度空间）的日志转换为wordpress可以导入的xml文件（包括所有的日志，评论，分类。图片暂不支持）

#### 对应Wordpress版本

2.5，其余版本未测试

#### 运行环境

PHP5 + icov模块，php4未测试

#### 使用方法

对文件做简单配置后运行即可

需要手动配置部分如下

```php
$start_page = 1; //起始页
$end_page = 17; //结束页，即blog的总页数 在blog点击 更多文章>>，再点击尾页，在这里填写尾页的页码
$baiduId = 'tsost'; //你的baidu空间名 例：我的空间url是http://hi.baidu.com/tsost，这里值就为tsost
```

#### 注意

尽管使用了自动刷新防止超时，但视网络情况仍然有可能发生超时，此时手动刷新即可。

程序运行会产生大量临时文件，建议单独放置一个文件夹

程序运行完毕会显示Baidu has been OOXX，运行结果为同目录下的export.xml

随便写的，没有严格按wp标准生成，在wp2.5下导入成功。当然也可以委托我负责转换，只需花费七街论坛1000RP，详见[这里](http://forum.ave7.net/showthread.php?t=18628)

[猛击我](http://cid-01e48df64f8bd957.skydrive.live.com/embedrowdetail.aspx/Source/ooxxbaidu.7z)下载

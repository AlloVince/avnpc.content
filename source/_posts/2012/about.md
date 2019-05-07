---
published: true
date: '2012-04-26 11:35:01'
tags: []
author: AlloVince
title: 关于我
---

AlloVince， 又名 Allo / 某 A / AV / 艾鲁， 网名源自“艾鲁维斯”。

西历 1984 年生人，已婚，经历过 80 后的典型童年、
[做过宅男](https://avnpc.com/pages/OX)、
[打过游戏](https://avnpc.com/pages/Farland_Series)、
[看过 YY 小说](https://avnpc.com/pages/memorialize_of_chinese_net_novels)、
[搞过 ACG](https://avnpc.com/pages/Memories_Off_2nd_ost_review)、
[听过同人音乐](https://avnpc.com/pages/shikata_akiko)、
[出过国](https://avnpc.com/pages/akihabara)、
[创过业](https://avnpc.com/pages/projects)。

07 年毕业后赴日本工作两年，主要负责某(大法)公司社内 ERP 系统，并在职场一线亲历了 07-08 年金融危机;

09-12 年于家乡兰州联合创办公司失败，但有幸认识了妻子;

13-15 年担任[华尔街见闻](https://wallstreetcn.com/)技术负责人开始第二次创业，帮助见闻从天使轮成长到 B 轮融资，技术团队从自己 1 人到离职前近 50 人;

16 年后使用 node.js 技术栈居多， 关注容器技术、机器学习。

## 出版作品

- [自制编程语言](http://book.douban.com/subject/25735333/) 2013
- [游戏开发的数学和物理](http://book.douban.com/subject/26274169/) 2014

## 开源项目

- [EveEngine.js](https://github.com/EvaEngine/EvaEngine.js) Node.js 的微服务开发引擎，帮助 Node.js 更快更规范的开始后端项目
- [EveEngine](http://avnpc.com/pages/eva-engine) 基于 Phalcon 框架的 PHP 快速开发引擎，同时也是华尔街见闻在职期间所有项目的底层，功能完备，性能优异，经实际项目考验，能以个位数台 VPS 支持数亿级别日请求并保持较低负载。
- [EvaOAuth](http://avnpc.com/pages/evaoauth) 统一接口设计的 OAuth Client，可以用相同的代码支持 OAuth1.0/OAuth2.0 网站，快速集成第三方用户体系。
- [EvaThumber](http://avnpc.com/pages/evathumber) 基于 URL 对图片进行缩放、水印、剪裁等常规操作。

## 技术分享 PPT

PPT 原稿保存于[我的 gist](https://gist.github.com/AlloVince)，请安装 node.js 后使用后文指令在本地打开查看

- 2018.5 [GraphQL 从入门到“放弃”](https://gist.github.com/AlloVince/8ba1c92890c74cc7f4e68f09c79ec0d1)
  - `npm install -g reveal-md && wget -q -O graphql.md https://gist.githubusercontent.com/AlloVince/8ba1c92890c74cc7f4e68f09c79ec0d1/raw/graphql.md && reveal-md graphql.md`
- 2015.7 [如何写出好的 PHP 代码](https://gist.github.com/AlloVince/a656e2842c7b6a43c81d)
  - `npm install -g reveal-md && wget -q -O php_code_review.md https://gist.githubusercontent.com/AlloVince/a656e2842c7b6a43c81d/raw/php_code_review.md && reveal-md php_code_review.md`
- 2015.4 [关于产品与项目的一些思考](https://gist.github.com/AlloVince/04dab3ad5c1f24c9faea)
  - `npm install -g reveal-md && wget -q -O project.md https://gist.githubusercontent.com/AlloVince/04dab3ad5c1f24c9faea/raw/project.md && reveal-md project.md`
- 2015.4 [RESTFul API 设计](https://gist.github.com/AlloVince/ba8c33138adbdd39d757)
  - `npm install -g reveal-md && wget -q -O restful.md https://gist.githubusercontent.com/AlloVince/ba8c33138adbdd39d757/raw/restfulapi.md && reveal-md restful.md`
- 2014.2 [Phalcon 的技术选型及优缺点评估](http://evaengine.github.io/EvaEngine/)


## 关于 avnpc.com

avnpc.com 从内容到设计、开发都是完全原创的，所有代码及内容都托管在 Github, 使用的主要技术包括：

- [API](https://github.com/AlloVince/avnpc.js)
  - [EveEngine.js](https://github.com/EvaEngine/EvaEngine.js) Node.js 的微服务开发引擎
  - [GraphQL-Boot](https://github.com/AlloVince/graphql-boot) 可能是 GraphQL 在 Node.js 下的最佳实践
- [FrontEnd](https://github.com/AlloVince/avnpc.front)
  - [Next.js](https://github.com/zeit/next.js) 服务端渲染(SSR)框架
  - [markdown-it](https://github.com/markdown-it/markdown-it) 可能是 JS 最好的 Markdown 渲染
  - [mermaid](https://mermaidjs.github.io/) 很方便的代码->图形的转换工具
  - [Ant Design](https://ant.design/) 阿里出品的 React 为主的 UI 解决方案
  - [gitalk](https://github.com/gitalk/gitalk) 使用 Github Issue 作为评论
- CI/Deployment
  - [semantic-release](https://github.com/semantic-release/semantic-release) 语义化发布, 适用于 npm 生态
  - [Travis CI](https://travis-ci.org/) CI 及自动构建打包发布
  - [Docker Hub](https://hub.docker.com/) 构建后的 Docker 镜像存储于此
  - [Docker Compose](https://docs.docker.com/compose/) Docker 编排工具
  - [Traefik](https://github.com/containous/traefik/) Docker 环境下好用的反向代理
- [Content](https://github.com/AlloVince/avnpc.content)

服务目前托管于 [Linode](https://www.linode.com/) ， 如果你也有意购买，可以使用我的 [Referral](https://www.linode.com/?r=a33af5735a21b63c784f7cd2cf87dba00fd319a2)

## 关于提问

老实说能在 Blog 上提问的朋友是出于信任我，我首先表示感谢，但事实上很多朋友并不懂得如何提问，主要症状包括：

1. 不分场合，随便找一篇文章问一些和当前文章本身不相干的事情。
2. 没有重点，不把问题的上下文和相关场景说清楚，直接贴一大段代码上来，你看着办。
3. 毫无思索，问一些 Google 一下立即能找到答案的问题，或者问题中没有自己的思考和看法，单纯将你当作人肉解题机。
4. 不讲礼貌，似乎有些朋友还没有弄清楚，别人并没有义务回答你的问题。

虽然本着礼貌的态度对于上面列举的一些问题也尽可能的做了回答，但如果你没有得到回答或者发现问题不见了，那么答案一般是“RTFM”/“RTFS”或者“STFW”。

希望没有阅读过《[提问的智慧](https://github.com/ryanhanwu/How-To-Ask-Questions-The-Smart-Way/blob/master/README-zh_CN.md)》一文的朋友，阅读一下这篇会让你终身受益的文章。

- 向我提问的正式方式： 在下面的技术问答网站邀请 AlloVince 回答问题
    - [Allo@StackOverflow](http://stackoverflow.com/users/1445934/allovince)
    - [Allo@SegmentFault](http://segmentfault.com/u/allovince)

## 联系方式

- 联系我： i@av2.me
- 关注我：
  - [Allo 的兴趣爱好][]
  - [Allo 的开源项目][]：欢迎 Follow / PR


[Allo 的兴趣爱好]: http://zh.wikipedia.org/wiki/User:AlloVince
[Allo 的开源项目]: https://github.com/AlloVince

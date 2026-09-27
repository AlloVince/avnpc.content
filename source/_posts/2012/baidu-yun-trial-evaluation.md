---
title: "百度开放云平台试用评测报告"
date: "2012-05-13 00:21:47"
slug: "baidu-yun-trial-evaluation"
published: true
author: "AlloVince"
legacy_id: 142
comment_status: "open"
comments: true
gitalk_id: "POST_142"
tags: []
---
−目录

- [照搬SAE的百度云环境（BAE）](#%E7%85%A7%E6%90%ACsae%E7%9A%84%E7%99%BE%E5%BA%A6%E4%BA%91%E7%8E%AF%E5%A2%83_bae)

- [极度坑爹的百度云数据库](#%E6%9E%81%E5%BA%A6%E5%9D%91%E7%88%B9%E7%9A%84%E7%99%BE%E5%BA%A6%E4%BA%91%E6%95%B0%E6%8D%AE%E5%BA%93)

- [差强人意的百度云存储（BCS）](#%E5%B7%AE%E5%BC%BA%E4%BA%BA%E6%84%8F%E7%9A%84%E7%99%BE%E5%BA%A6%E4%BA%91%E5%AD%98%E5%82%A8_bcs)

- [有点意思的云消息(BCMS)](#%E6%9C%89%E7%82%B9%E6%84%8F%E6%80%9D%E7%9A%84%E4%BA%91%E6%B6%88%E6%81%AF_bcms)

- [总结](#%E6%80%BB%E7%BB%93)

[百度开放云平台](http://yun.baidu.com/ "http://yun.baidu.com/")终于做了一次大的整合，目前只能通过邮件申请邀请码，申请方法参看官方的[如何获取百度云平台邀请码](http://dev.baidu.com/wiki/yun)

我是在周五下午16：0发出了申请邮件，16：09收到邀请码，神速度！

试用开始，我并没有非常深入的测试，只是就目前后台看到的部分和已经公开的文档说明拍些砖。

目前百度云只开放了云环境（BAE）、云存储（BCS）、云消息(BCMS)、云数据库几项，最为重头戏的虚拟机(BVM)则还没有开放。

# 照搬SAE的百度云环境（BAE）

[百度云环境（BAE）](/pages/baidu-yun-trial-evaluation "http://avnpc.com/pages/baidu-yun-trial-evaluation")基本几乎照搬了新浪SAE的模式，表现为：

- 语言为PHP，java还在测试中

- 版本管理仅支持SVN。

- 对PHP文件操作等函数做了大量限制

- 对cURL则做了一层封装

其中版本管理还使用SVN而不是GIT让我觉得有些诧异，考虑到SAE的上线时间，在当时用SVN无可厚非。但在GIT如日中天的今天，连Google Code都提供了GIT支持，BAE还在固守SVN的话，实在不利于开发调试和多项目沟通部署。

当然，完全可以把BAE作为一个私有的SVN仓库，在速度方面是很不错的。

# 极度坑爹的百度云数据库

看到[百度云数据库](/pages/baidu-yun-trial-evaluation "http://avnpc.com/pages/baidu-yun-trial-evaluation")居然用了MySQL的一瞬间，我就有不好的预感，果然云数据库是百度云平台里最坑爹的一项。

百度云数据库的最大问题就是，仅仅提供了phpmyadmin作为管理后台，**完全没有考虑过数据迁移的问题**。

哪怕仅仅是个人博客，可能累计一段时间以后的数据量也会超过5MB，而通过百度云平台提供的phpmyadmin，只允许用户上传最大5M的文件导入数据。

也就是说，只要项目的数据超过5MB，就没有什么好办法能将以前的项目的数据迁移到百度云数据库，难道要为了迁移数据专门写个程序？mysqlDump表示很桑心。

更不要说百度云数据库还阉割掉了适用于分布式的Cluster引擎，单库最大5GB的限制，完全丧失了可用性。

作为一个超过5M的数据都没法迁移就敢上线测试的数据库产品来说，请允许我对百度云数据库的产品经理表示崇高的敬意。

# 差强人意的百度云存储（BCS）

[百度云存储（BCS）](/pages/baidu-yun-trial-evaluation "http://avnpc.com/pages/baidu-yun-trial-evaluation")是唯一让我没有太大意见的部分，只是目前没有提供任何文件的上传工具，如果说从易用性考虑，至少应该开放ftp上传，否则文件一多还要单独开发批量上传的工具，成本太高。

百度云存储的一个标准Bucket可以达到20G容量，每月60G带宽+200W请求数，还是很让人动心的。同样作为文件存储的收费项目又拍云有没有感到压力呢。

另外有v2ex童鞋测试发现[文件的Content-Type并不准确](http://www.v2ex.com/t/17751 "http://www.v2ex.com/t/17751")，算是一个小问题。

# 有点意思的云消息(BCMS)

[百度云消息(BCMS)](/pages/baidu-yun-trial-evaluation "http://avnpc.com/pages/baidu-yun-trial-evaluation") 一开始让我误以为是消息队列，后来才发现是可以发送免费邮件和免费手机短信的服务，对于开发手机短信验证这样的小功能来说绝对是个福音，省去再去找不靠谱的SP的麻烦。

每月1000短信数和10000邮件数的配额对于中小型的应用算是基本够了，只是不知道邮件的在主流邮箱的垃圾邮件识别率如何。

# 总结

百度云整体来看还是算有几分诚意的，特别是一开始配额方面给的比较慷慨。

但其实综合考量下来，用目前的百度云开发应用仍然不够靠谱，至少恶心的云数据库一环就消灭99%的迁移项目。

可以持观望态度看看后续的定价策略和产品改进。有兴趣的可以做点小东西玩，至于开发大型应用，还是洗洗睡吧。

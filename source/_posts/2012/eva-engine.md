---
slug: eva-engine
published: true
date: '2012-05-14 17:02:35'
tags:
  - EvaEngine
author: AlloVince
title: EvaEngine Project
---

什么是EvaEngine
=============================================

EvaEngine是一个基于PHP的开发引擎。出于对性能的考虑，底层框架从Zend Framework2更换为[Phalcon Framework](http://phalconphp.com/en/) 1.3.X。目前EvaEngine 0.1.X版本已经稳定运行在千万级别PV的[生产环境](http://wallstreetcn.com/)5个月，很可能已经是中国最大的使用Phalcon Framework的案例。EvaEngin裸引擎性能在4核4G内存的测试机器上超过400QPS。欢迎对项目贡献代码或者提出意见。

EvaEngine正在努力实现以下的目标：

- 让业务增长平滑过渡，从单台VPS到大型分布式架构，只需要改变引擎的部署方式
- 高性能，得益于C扩展框架的性能提升，能够以极少的机器支持千万甚至上亿级别的访问量
- 真正意义上的模块化，简化项目开发中70%的重复工作。

EvaEngine将作为开源项目存在，许可证将采用与Zend一致的New BSD License

项目源代码：

[https://github.com/EvaEngine/EvaEngine](https://github.com/EvaEngine/EvaEngine)



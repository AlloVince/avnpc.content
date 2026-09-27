---
title: "ZF2重大变更：在MVC中去除Bootstrap，引入ServiceManager"
date: "2012-05-18 14:45:35"
slug: "zf2-new-changes-of-servicemanager"
published: true
author: "AlloVince"
legacy_id: 145
comment_status: "open"
comments: true
gitalk_id: "POST_145"
tags: []
---
Zend2这两天做了一次巨大的改动，将ServiceManager合并入主干，同时在MVC构架中彻底去掉了原来的启动管理Zend/Mvc/Bootstrap.

这个改动几乎颠覆了整个Zend2.0至今为止的所有的已有Demo和模块，是一次具有破坏性的改动。（[Evan Coury原话](http://zend-framework-community.634137.n4.nabble.com/ServiceManager-MVC-integration-merged-with-master-BC-breaking-changes-td4643952.html)）

在ZF2，Bootstrap所负责的任务已经非常单薄了，核心功能简单说就是提供了一组Di的默认配置，然后按照顺序启动Mvc，并且触发Mvc事件。

ServiceManager引入并且去掉Bootstrap的意义，则在于将Mvc启动过程中的所有Di注入完全托管给用户，从而进一步简化构架，提高效率。

另外可能带来的直接好处是Di的配置能得到一定程度的简化。

抛开技术上的问题不谈，从使用上来说，这次的变动让升级的成本变得很高，而最终版的ZF2，私以为还是需要原来的Zend/Mvc/Bootstrap这样的默认设置，否则带给开发者的学习成本也太高。

最近的ZF2对[Zend\Form](http://zend-framework-community.634137.n4.nabble.com/Forms-now-in-master-td4637456.html)也终于进行了全面的改写，实现方法比原来灵活了很多，终于具备了一定的实用性，可喜可贺。

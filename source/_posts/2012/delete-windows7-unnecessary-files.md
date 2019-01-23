---
published: true
date: '2012-10-22 12:24:40'
tags:
  - Windows7
author: AlloVince
title: 记录Windows7减肥瘦身过程
---

周末重装了一次系统，在我的Thinkpad T420s下 Window7 Professional + SP1补丁 + Thinkpad驱动更新至最新。还没有装任何软件，磁盘占用已经30G+了。简直是叔叔可忍婶婶也不能忍啊！

经过简单的减肥瘦身，最终把Win7控制在18.5G,还算勉强可以接受，记录过程如下：

1. 运行磁盘清理

2. 关闭系统保护

```
计算机——属性——系统保护——配置——关闭系统保护
```

3. 删除自动更新下载文件

```
C:\Windows\SoftwareDistribution\Download 下所有文件
```

4. 删除SP1备份文件

```
开始——运行——cmd
dism /online /cleanup-image /spsuperseded
```

5. 关闭休眠文件，cmd下

```
powercfg -h off
```

6. 压缩windows驱动文件夹

```
C:\Windows\winsxs
属性——高级——压缩内容以已节省磁盘空间
```

7. 删除Thinkvantage System Update下载文件

```
C:\Program Files (x86)\Lenovo\System Update\session 除了system和temp以外文件夹全部删除
```

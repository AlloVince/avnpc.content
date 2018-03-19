---
slug: recovery-data-from-corrupted-sd-card
published: true
date: '2012-10-16 23:16:28'
tags: []
author: AlloVince
title: 记一次乱码SD卡损坏数据的恢复过程
---

记录一次[SD卡乱码的数据恢复](http://avnpc.com/pages/recovery-data-from-corrupted-sd-card)的过程，起因大概是不靠谱的读卡器+没有正确的拔SD卡，再次读SD卡发现里面的文件和文件夹已经都变成了乱码，文件夹无法打开，重命名则提示拒绝访问。在磁盘驱动器界面仍然能显示出容量被占用。

Google了一番，按照最常见的修复方式是，右键单击SD卡分区，属性->工具->修复->勾选“自动修复文件系统错误”。但是发现执行完毕时候，乱码虽然不见了，原来的文件夹却变成了一个无扩展名的文件。

此时为了尽可能保护数据，使用Ghost对SD卡做了一次镜像备份。Ghost的Win32版本可以直接在Windows下操作。流程为：

Local -> Partition -> To Image

选择SD卡（一般是容量数字小的一个），继续选择分区（一般只有一个） -> OK -> 填写文件名并保存，如 SD.gho -> 选择No （不压缩）

最后会生成一个SD.gho

此时用Ghost Explorer打开gho文件，就可以看到里面已经找到的文件，将其复制出来就可以了。

我常用的[Ghost下载](http://www.bego.cc/file/10053318)

记录下来整个解决过程，希望对碰到同样问题的朋友有帮助。

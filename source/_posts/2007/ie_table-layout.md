---
title: "有关RP的table-layout以及表格文字溢出问题小结"
date: "2007-04-12 20:17:00"
slug: "ie_table-layout"
published: true
author: "AlloVince"
legacy_id: 74
comment_status: "closed"
comments: false
tags:
  - "css"
  - "YD的程序员葛阁"
---
**目录** [折叠]

1. [Step.1](#toc_96731100)
2. [Step.2](#toc_96750000)
3. [Step.3](#toc_96755300)

目的很简单，就是让表格保持固定大小，单元格的长、宽都固定，不被单元格内的文字溢出改变布局

#### Step.1

固定表格，这里用到了RP的table-layout的fix属性

手册里是这样的：

> 版本：CSS2　 兼容性：IE5+　继承性：有
>
> auto : 　默认的自动算法。布局将基于各单元格的内容。表格在每一单元格读取计算之后才会显示出来。速度很慢
>
> fixed : 　固定布局的算法。在这算法中，水平布局是仅仅基于表格的宽度，表格边框的宽度，单元格间距，列的宽度，而和表格内容无关
>
> 说明： 设置或检索表格的布局算法。

简单说，就是将表格的单元格布局钉死。

#### Step.2

理论上，接下来只需要在td上加上overflow:hidden，让溢出的文字不显示就万事大吉了，但事实上远没有那么简单，table-layout 有个很RP的问题，如果表格内的文字出现空格、中文字符等时，会自动进行换行，这样一来宽度是有保证了，高度仍然会溢出。

这里首先考虑用nowrap，目的是使单元格强制不换行。但事实上nowrap也是个很RP的属性，它只有在单元格未指定宽度的情况下才是有用的。我们的目的是固定单元格的长宽，显然单在td后面加上nowrap也是行不通的。

#### Step.3

那么就只能把很简单的事情搞到如此复杂了……

```xml
<table width="100" style="table-layout:fixed;">
  <tr>
    <td width="100" style="overflow:hidden;text-overflow:ellipsis; white-space:nowrap;"><div nowrap>这里是一个测试，再多的汉字和空格也不怕</div></td>
  </tr>
</table>
```

text-overflow是一个很好css，可以把溢出的文字用...截断并提示。事实上我想说的是，布局的时候尽可能用div吧，这种RP的问题用div的话一句话就完了。。。

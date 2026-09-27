---
title: "基于Tag标签的简单三表查询相关度实现"
date: "2009-02-12 13:44:03"
slug: "tag-based_simple_correlation"
published: true
author: "AlloVince"
legacy_id: 126
comment_status: "closed"
comments: false
tags:
  - "AVBlog"
  - "SQL"
  - "Tag"
  - "YD的程序员葛阁"
  - "相关度"
---
AVBlog2.0开发手记之2

在以往的相关度实现中，都需要实现搜索引擎的类似功能，对检索的所有内容进行分词、索引，然后统计对比\[\[http://zh.wikipedia.org/wiki/TF-IDF|TF/IDF]]，从而对内容的相关性作出判断。关于上述内容，Google研究员吴军的[数学之美系列中有一篇很好的文章](http://www.googlechinablog.com/2006/07/12.html)可供参考。

而Tag的出现，是对相关度实现的一个绝好补充。因为以往由机器以统计方式获得的关键词，改由用户作出人为的描述，准确性和适用性都是机器所不能比的。

在一个简单系统中，往往采用一个内容表，一个Tag表，一个关联表这样的多对多关系来实现Tag功能。例如这样的三个表分别是内容表Blogs，标签表Tags和关联表Marks，Blogs/Tags都有主键id，关联表通过blog_id和tag_id将三个表联系起来。

```sql
CREATE TABLE `blogs` (
  `id` int(10) auto_increment,
  `title` varchar(255) ,
  PRIMARY KEY  (`id`)
);

CREATE TABLE `tags` (
  `id` int(10) auto_increment,
  `tag` varchar(100) ,
  `count` int(10),
  PRIMARY KEY  (`id`)
);

CREATE TABLE `marks` (
  `tag_id` int(10),
  `blog_id` int(10),
  PRIMARY KEY  (`tag_id`,`blog_id`)
) ;
```

那么在这样的系统中，如果想要得到文章A的关联内容，只需要检索文章A的所有标签，然后在检索这些标签所关联的文章就可以了。例如我们想要获得id为1的blog的关联blog，可以在Marks表中用[left join](http://en.wikipedia.org/wiki/Join_\(SQL\))这样实现

```sql
SELECT B.blog_id, B.tag_id
FROM marks AS A
LEFT JOIN marks AS B ON A.tag_id = B.tag_id
WHERE A.blog_id =1 and B.blog_id != 1
```

结果栏的blog_id就是所有id为1的相关blog

这样做存在很多缺点，首先这是在认为所有标签的重要性都一样的前提下的结果。但事实上Tag自身也有着不同的权重，例如在我的Blog中，【Javascript】这个标签有12篇文章被标记，【Apache】这个标签只有两篇文章被标记。如果每篇文章都只有一个标签的话，显然被标记【Apache】的两篇文章要比标记【Javascript】的12篇文章联系更加紧密。

于是我们可以在这里借用TF/IDF的概念来计算一个Tag的权重，如果Blog的数量有100篇，那么【Javascript】的Tag就是Ln(100/12)=2.120，【Apache】则是Ln(100/2)=3.912。体现到SQL语句里就是这样

```sql
SELECT B.blog_id, B.tag_id, LOG( 100 / C.count ) AS weight
FROM marks AS A
LEFT JOIN marks AS B ON A.tag_id = B.tag_id
LEFT JOIN tags AS C ON B.tag_id = C.id
WHERE A.blog_id =1
AND B.blog_id !=1
```

通过关联Tag表以及Tag表的count值(即为一个Tag标记的Blog数)获得Tag的权重。

接下来的问题是，一个文章往往会关联多个Tag，例如

- 文章A的Tag有【Javascript】、【Apache】、【Css】、【Ajax】
- 文章B的Tag有【Javascript】、【Apache】、【Google】
- 文章C的Tag有【Json】、【Apache】、【XML】

如果Tag的权重都一样的话，显然这三篇文章的相关度是AB>AC的。因为AB有两个标签一致，而AC只有一个。

如何体现在SQL联合查询上呢，我们只需要对上面做少量修改

```sql
SELECT B.blog_id, B.tag_id, SUM( LOG( 100 / C.count ) ) AS weight
FROM marks AS A
LEFT JOIN marks AS B ON A.tag_id = B.tag_id
LEFT JOIN tags AS C ON B.tag_id = C.id
WHERE A.blog_id =1
AND B.blog_id !=1
GROUP BY B.blog_id
ORDER BY weight DESC
```

这里利用了SUM函数和GROUP BY语句。

至此我们通过1个SQL语句就获得了按照相关度排序的序列，虽然使用了两次比较消耗资源的Left Join， 不过因为关联表都是数字序列，操作又集中在主键和索引上，还是可以放心使用的。

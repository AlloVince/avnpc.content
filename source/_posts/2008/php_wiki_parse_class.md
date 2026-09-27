---
title: "PHP Wiki语法解析类"
date: "2008-07-18 13:05:19"
slug: "php_wiki_parse_class"
published: true
author: "AlloVince"
legacy_id: 103
comment_status: "closed"
comments: false
tags:
  - "AV Framework"
  - "FrameWork"
  - "php"
  - "WIKI"
---
出于很多考虑，终于还是扔掉了[Pear](http://pear.php.net/)的[Text_wiki](http://pear.php.net/package/Text_Wiki/redirected)改用自己写的Wiki语法解析，专用于本[Framework](av_framework)。比起Text_wiki来改善，或者说是按自己的习惯改变了很多方面:

1. 语法上综合了[wikipedia](http://wikipedia.org/)和[puki wiki](http://c2.com/cgi/wiki?PukiWiki)。
2. 支持html标记插入。
3. 生成的代码通过W3C Strict验证。
4. 即使不做解析，也是一篇表意良好，格式工整的text文档，而不像很多乍看上去如天书一般的Wiki语法
5. 解析速度比Text_wiki快
6. 加入了很多方便的特殊标记。

目前只是发布前的演示，正式版将附加在Framework中发布，有兴趣的同学可以索取源代码。那么，实验开始：

### Title Test

Wiki代码：

```
=h1=
==h2==
===h3===
====h4====
=====h5=====
======h6======
=======h7=======
```

转换结果：

# h1

## h2

### h3

#### h4

##### h5

###### h6

###### =h7=

---

### Table Test

Wiki代码：

```
! 1 ! 2 ! 3 ! 4 ! 5 !
! 1 | 2 | 3 | 4 | 5 |
! 1  2 !| 3 | 4 | 5 |
! 1  2 || 3 | 4 | 5 |
! 1 | 2  3 || 4 | 5 |
! 1 | 2 | 3  4 || 5 |
! 1 | 2 | 3 | 4  5 ||
! 1  2  3 !!| 4 | 5 |
! 1 | 2  3  4 ||| 5 |
! 1 | 2 | 3  4  5 |||
! 1  2  3  4 !!!| 5 |
! 1 | 2  3  4  5 ||||
! 1  2  3  4  5 |||||

! ---------- ! ----------- ! ---------- !
!left        |        right|   middle   |
```

转换结果：

| 1 | 2 | 3 | 4 | 5 |
| :-: | :-: | :-: | :-: | :-: |
| **1** | 2 | 3 | 4 | 5 |
| **1 2** |  | 3 | 4 | 5 |
| 1 2 |  | 3 | 4 | 5 |
| **1** | 2 3 |  | 4 | 5 |
| **1** | 2 | 3 4 |  | 5 |
| **1** | 2 | 3 | 4 5 |  |
| **1 2 3** |  |  | 4 | 5 |
| **1** | 2 3 4 |  |  | 5 |
| **1** | 2 | 3 4 5 |  |  |
| **1 2 3 4** |  |  |  | 5 |
| **1** | 2 3 4 5 |  |  |  |
| 1 2 3 4 5 |  |  |  |  |

| ---------- | ----------- | ---------- |
| :-: | :-: | :-: |
| **left** | right | middle |

---

### Paragraph Test

Wiki代码：

```
//Remark will not be parse
This is a normal paragraph.

This is another normal paragraph.

This is a paragraph with newline of br mark.
This is a paragraph with newline of br mark.
This is a paragraph with newline of br mark.
```

转换结果：

This is a normal paragraph.

This is another normal paragraph.

This is a paragraph with newline of br mark.  
This is a paragraph with newline of br mark.  
This is a paragraph with newline of br mark.

---

### Blockquote Test

Wiki代码：

```
> Blockquote Level1
>> Blockquote Level2
>> Blockquote Level2
>>> Blockquote Level3
>>> Blockquote Level3
```

转换结果：

> Blockquote Level1
>
> > Blockquote Level2
> >
> > Blockquote Level2
> >
> > > Blockquote Level3
> > >
> > > Blockquote Level3

---

### List Test

Wiki代码：

```
* ul list level1
** ul list level2
** ul list level2
*** ul list level3
**** ul list level4
**** ul list level4
* ul list level1
** ul list level2
* ul list level1

# ol list level1
## ol list level2
## ol list level2
### ol list level3
#### ol list level4
#### ol list level4
# ol list level1
## ol list level2
# ol list level1
```

转换结果：

- ul list level1
  - ul list level2
  - ul list level2
    - ul list level3
      - ul list level4
      - ul list level4
- ul list level1
  - ul list level2
- ul list level1

1. ol list level1
   1. ol list level2
   2. ol list level2
      1. ol list level3
         1. ol list level4
         2. ol list level4
2. ol list level1
   1. ol list level2
3. ol list level1

---

### Advance Test

Wiki代码：

```
Wiki Parser can identify a url [http://allo.ave7.net/] or an image [http://www.ave7.net/logo.gif] with [http://allo.ave7.net/|account].
You can also use <i><b>Html Marks</b></i> inline.
How forcibly specify this is a url [link:http://www.ave7.net/logo.gif] or an image
[img:http://allo.ave7.net/photo/picasa/allovince/5220926002522558673/5220926248939178194|photo].
```

转换结果：

Wiki Parser can identify a url [http://allo.ave7.net/](http://allo.ave7.net/) or an image ![http://www.ave7.net/logo.gif](http://www.ave7.net/logo.gif) with [account](http://allo.ave7.net/).  
You can also use ***Html Marks*** inline.  
How forcibly specify this is a url [http://www.ave7.net/logo.gif](http://www.ave7.net/logo.gif) or an image  
![photo](http://allo.ave7.net/photo/picasa/allovince/5220926002522558673/5220926248939178194).

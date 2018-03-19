---
slug: import-export-drupal-data-by-restful-service
published: true
date: '2013-07-19 15:41:39'
tags:
  - Drupal
  - RESTFul
author: AlloVince
title: 实战Drupal之通过RESTFul Service实现Drupal数据的导入与导出
---

如果有第三方系统需要整合Drupal，或者用Drupal来整合其他系统，难以避免的会有数据的导入与导出。我们一般有两种实现方法：

1. 创建一个Drupal模块，通过Drupal内置函数写入/读取数据。
2. 为Drupal搭建一套WebService API，通过http方式进行数据的交互。

从性能来讲，前者肯定是占优的，但是需要对Drupal有更深入的了解。后者虽然牺牲了一些性能，但是有更好的适用性，应用范围更广。下文将介绍[Drupal 7下RESTFul服务的搭建以及如何实现数据的导入与导出](http://avnpc.com/pages/import-export-drupal-data-by-restful-service)。


## Drupal搭建RESTFul Service

Drupal可以通过以下几个模块简单的搭建出一个高度可定制的RESTFul Service：

1. [Views](https://drupal.org/project/views/) 可以说是Drupal最强大的一个插件，并且已经成为Drupal8的核心插件了。通过Views可以在后台将Drupal的数据定制整合输出，可以视为一个简易的可视化数据库管理器。
2. [Services](https://drupal.org/project/Services) 将Drupal数据公开为多种格式的API，包括XMLRPC、JSON、JSONP等。
3. [Services Views](https://drupal.org/project/services_views) 桥接Views与Services模块，让Views中的数据也可以通过Service输出。
4. [Views content cache](https://drupal.org/project/views_content_cache) Views默认的缓存太简单，通过Views content cache可以实现基于内容控制的缓存。

## 开放一个Drupal的自定义RESTFul API

安装Services模块后，首先需要通过 Home › Administration › Structure › Services › Add 添加一个Service，模块规定每个Service必须有`endpoints`参数，可以理解为URL的前缀部分。同时如果没有安装其他Service认证模块的话，**务必需要勾选Session authentication**，否则将无法写入数据。

比如域名为avnpc.com，如下填写：

- Machine-readable name of the endpoint : drupalapi
- Server : REST
- Path to endpoint : drupalapi
- Session authentication : 勾选

保存后可以看到Services列表中已经出现了一组名为drupalapi的Service。可以通过Edit Resources来选择具体需要公开哪些资源。以node为例，公开node资源后访问：

```
GET http://avnpc.com/drupalapi/node
```

即可以得到XML格式的node列表：

```
<result is_array="true"><item><nid>49038</nid><vid>49063</vid><type>news</type><language>zh-hans</language>...
```

Drupal Services还支持通过更改资源扩展名切换资源格式，如

```
GET http://avnpc.com/drupalapi/node.json
```

将得到JSON格式的node列表

```
[{"nid":"49038","vid":"49063","type":"news","language":"zh-hans"...
```

更加完整的Drupal RESTFul API可以参考[Drupal Services官方文档](https://drupal.org/node/1699354)。


### 通过资源Actions获得附属/关联资源

Drupal Services中还存在Actions的概念。这里的Actions可以理解为对于附属/关联资源提供的接口。

比如在Drupal中创建了一个分类（Taxonomy）下的Item：

- Name ： 中国
- Id ：7351

那么默认可以通过

```
GET http://avnpc.com/drupalapi/taxonomy_term/7351
```

获得该Item的信息

```
<result><tid>7351</tid><vid>2</vid><name>中国</name>...
```

而往往我们还需要获得该分类下的所有node，此时就需要使用taxonomy下的`selectNodes` Action来获取数据。

但是Drupal Services设计很奇怪的地方是Actions一般需要用POST方法获得，这很难还能称得上是“RESTFul”风格，比如我理解获得一个taxonomy item下的node理论上应该是这样：

```
GET http://avnpc.com/drupalapi/taxonomy_term/7351/selectNodes
```

但实际上我们需要这样做：

```
POST http://avnpc.com/drupalapi/taxonomy_term/selectNodes
```

POST header中需要包含：

```
Content-Type: application/x-www-form-urlencoded
```

POST body为：

```
tid=7351
```

实在让人无法言语。所以一般来说比较复杂的资源，还是需要通过Views + Services Views共同实现比较好。


## 通过Drupal Views创建可以接收参数的RESTFul API

可能通过一个实例来说明最清楚不过了，比如在Drupal中定义了一个Content Type为news（新闻），新闻关联了若干自定义字段，包括：

- 新闻分类 field_category 一个新闻可以对应多个目录
- 新闻格式 field_format

等等，其中新闻分类field_category又关联到一个已有的类别（taxonomy）。这个例子是一个非常常见的多对多结构。

而此时我们希望开放一个API，形如：

```
GET http://avnpc.com/drupalapi/news?tid=<tid>
```

可以通过更改URL中的`tid`参数，将该分类下的新闻全部筛选出来。操作方法如下：

首先还是要确认一下Services Views模块是否已经安装

__1. 添加一个新的视图（View） Home › Administration › Structure › Views › Add new view__

- View name : news
- Show : Content
- of type : 新闻
- 勾选 Create a block 
- Continue & edit

__2. 在View中创建Service，设置路径__

- Add -> 选择Services

可以看到View中新建了一个Services标签页

- Services settings -> Path -> 设置为 news

__3. 设置外部可以更改的参数__

- Filter criteria -> Add
- 勾选 Content: Has taxonomy term -> Apply
- 选择要关联的Taxonomy Vocabulary ， Selection type 建议为Dropdown -> Apply & continue
- 勾选  Expose this filter to visitors, to allow them to change it 
- 在UI最下方新出现的More中， 设置Filter identifier为`tid`
- Apply

__4. 保存并公开Service API__

保存这个View，在Home › Administration › Structure > drupalapi中已经可以看到刚才新建的news。勾选将其公开出去。

然后可以尝试访问一下

```
GET http://avnpc.com/drupalapi/news?tid=7351
```


### 解决Services Views产生的缓存不更新BUG

经过测试，通过上面方法公开的Views API是无法根据URL参数来设置不同缓存的，有人已经给出了[解决Views Cache问题的Patch](https://drupal.org/node/1055616#comment-7034686)但是似乎还没有正式合并入发布分支，只能自己先打上补丁了。

下载补丁后放在views模块下，运行

    patch -pNUM views_1055616_81_cache.patch

即可


## Drupal RESTFul API创建/更新资源

通过上面的方法，已经可以将Drupal的所有资源以RESTFul API的形式公布给第三方用户，从而可以实现数据的导出或与第三方的交互。而如果要向Drupal中导入数据，则需要访问API的写入接口。

下面为了更好说明，URL请求部分采用[Requests](https://github.com/rmccue/Requests)来实现。

### 权限认证

如果没有安装其他第三方认证模块，Drupal RESTFul的权限认证默认是通过Session来进行的：

``` php
Requests::post('http://avnpc.com/apiv1/user/login.json', array(
    'Content-type' => 'application/json'
), json_encode(array('username' => 'yourname', 'password' => '123456')));
```

这样可以取得一个`session_name`和`sessid`，之后的写入请求需要将这两部分用`=`拼接后附加在header的`Cookie`中即可。

但是由于旧版本的Drupal Services还存在[CSRF](https://drupal.org/node/1890222)安全漏洞，最新的Drupal Services中还加入了`X-CSRF-Token`的限制。需要携带登陆后的Session信息去换取一个CSRF Token：

``` php
Requests::post('http://avnpc.com/services/session/token', array(
    'Cookie' => 'session info here',
));
```

那么完整的权限认证代码如下：

``` php
$res = Requests::post('http://avnpc.com/apiv1/user/login.json', array(
    'Content-type' => 'application/json'
), json_encode(array('username' => 'yourname', 'password' => '123456')));

$loginRes = json_decode($res->body);

$token = Requests::post('http://avnpc.com/services/session/token' array(
    'Cookie' => $loginRes->session_name . '=' . $loginRes->sessid,
));
$loginRes->token = $token->body;
```

最后就可以使用认证信息去创建资源了:

``` php
Requests::post('http://avnpc.com/apiv1/node', array(
    'Cookie' => $loginRes->session_name . '=' . $loginRes->sessid,
    'X-CSRF-Token' => $loginRes->token,
    'Content-type' => 'application/json'
), json_encode(array(
    //node info array
)));
```


### 如何通过Drupal RESTFul API创建附带自定义字段的资源

Drupal的最大优势就是可以灵活的为Node绑定任意自定义字段，那么在通过RESTFul API创建资源时，也避免不了会有自定义字段的写入。

还是以上文中的news资源为例，创建一个news资源的代码如下：

``` php
$obj = new stdClass();
$obj->title = '新闻标题‘;
$obj->status = 1;  //1 published
$obj->comment = 2; //1 close, 2 open
$obj->type = 'news';
$obj->language = 'zh-hans';
$obj->date = date('Y-m-d H:i:s +0000', mktime());

$body = new stdClass();
$body->value = '新闻内容';

$obj->body = new stdClass();
$obj->body->und[] = $body;

$obj->field_category = new stdClass();
$category = array(
    '7351' => '7351',
    '7352' => '7352',
);
$obj->field_category->und =  $category;

$obj->field_format = new stdClass();
$obj->field_format->und = new stdClass();
$obj->field_format->und->value = 'bold';
Requests::post('http://avnpc.com/apiv1/node', array(
    'Cookie' => $loginRes->session_name . '=' . $loginRes->sessid,
    'X-CSRF-Token' => $loginRes->token,
    'Content-type' => 'application/json'
), json_encode($obj));
```

可能最重要的部分就是让POST提交的数据结构保持与自定义node字段的结构完全一致。那么如何获得一个完整的Node创建时所需要的数据结构？一个非常简单的办法是找到node模块下的node.module。对`node_submit($node)`的`$node`设置断点，然后进入后台创建一个node，就可以看到node创建所需要的完整数据结构了。


## 参考

- [Services 3 - POST node.create with custom fields](https://drupal.org/node/1354202)



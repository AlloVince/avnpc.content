---
published: true
date: '2013-07-19 15:41:39'
tags:
  - Drupal
  - RESTFul
author: AlloVince
title: 实战 Drupal 之通过 RESTFul Service 实现 Drupal 数据的导入与导出
---

如果有第三方系统需要整合 Drupal，或者用 Drupal 来整合其他系统，难以避免的会有数据的导入与导出。我们一般有两种实现方法：

1. 创建一个 Drupal 模块，通过 Drupal 内置函数写入/读取数据。
2. 为 Drupal 搭建一套 WebService API，通过 http 方式进行数据的交互。

从性能来讲，前者肯定是占优的，但是需要对 Drupal 有更深入的了解。后者虽然牺牲了一些性能，但是有更好的适用性，应用范围更广。下文将介绍[Drupal 7 下 RESTFul 服务的搭建以及如何实现数据的导入与导出](http://avnpc.com/pages/import-export-drupal-data-by-restful-service)。


## Drupal 搭建 RESTFul Service

Drupal 可以通过以下几个模块简单的搭建出一个高度可定制的 RESTFul Service：

1. [Views](https://drupal.org/project/views/) 可以说是 Drupal 最强大的一个插件，并且已经成为 Drupal8 的核心插件了。通过 Views 可以在后台将 Drupal 的数据定制整合输出，可以视为一个简易的可视化数据库管理器。
2. [Services](https://drupal.org/project/Services) 将 Drupal 数据公开为多种格式的 API，包括 XMLRPC、JSON、JSONP 等。
3. [Services Views](https://drupal.org/project/services_views) 桥接 Views 与 Services 模块，让 Views 中的数据也可以通过 Service 输出。
4. [Views content cache](https://drupal.org/project/views_content_cache) Views 默认的缓存太简单，通过 Views content cache 可以实现基于内容控制的缓存。

## 开放一个 Drupal 的自定义 RESTFul API

安装 Services 模块后，首先需要通过 Home › Administration › Structure › Services › Add 添加一个 Service，模块规定每个 Service 必须有`endpoints`参数，可以理解为 URL 的前缀部分。同时如果没有安装其他 Service 认证模块的话，**务必需要勾选 Session authentication**，否则将无法写入数据。

比如域名为 avnpc.com，如下填写：

- Machine-readable name of the endpoint : drupalapi
- Server : REST
- Path to endpoint : drupalapi
- Session authentication : 勾选

保存后可以看到 Services 列表中已经出现了一组名为 drupalapi 的 Service。可以通过 Edit Resources 来选择具体需要公开哪些资源。以 node 为例，公开 node 资源后访问：

```plain
GET http://avnpc.com/drupalapi/node
```

即可以得到 XML 格式的 node 列表：

```plain
<result is_array="true"><item><nid>49038</nid><vid>49063</vid><type>news</type><language>zh-hans</language>...
```

Drupal Services 还支持通过更改资源扩展名切换资源格式，如

```plain
GET http://avnpc.com/drupalapi/node.json
```

将得到 JSON 格式的 node 列表

```plain
[{"nid":"49038","vid":"49063","type":"news","language":"zh-hans"...
```

更加完整的 Drupal RESTFul API 可以参考[Drupal Services 官方文档](https://drupal.org/node/1699354)。


### 通过资源 Actions 获得附属/关联资源

Drupal Services 中还存在 Actions 的概念。这里的 Actions 可以理解为对于附属/关联资源提供的接口。

比如在 Drupal 中创建了一个分类（Taxonomy）下的 Item：

- Name ： 中国
- Id ：7351

那么默认可以通过

```plain
GET http://avnpc.com/drupalapi/taxonomy_term/7351
```

获得该 Item 的信息

```plain
<result><tid>7351</tid><vid>2</vid><name>中国</name>...
```

而往往我们还需要获得该分类下的所有 node，此时就需要使用 taxonomy 下的`selectNodes` Action 来获取数据。

但是 Drupal Services 设计很奇怪的地方是 Actions 一般需要用 POST 方法获得，这很难还能称得上是“RESTFul”风格，比如我理解获得一个 taxonomy item 下的 node 理论上应该是这样：

```plain
GET http://avnpc.com/drupalapi/taxonomy_term/7351/selectNodes
```

但实际上我们需要这样做：

```plain
POST http://avnpc.com/drupalapi/taxonomy_term/selectNodes
```

POST header 中需要包含：

```plain
Content-Type: application/x-www-form-urlencoded
```

POST body 为：

```plain
tid=7351
```

实在让人无法言语。所以一般来说比较复杂的资源，还是需要通过 Views + Services Views 共同实现比较好。


## 通过 Drupal Views 创建可以接收参数的 RESTFul API

可能通过一个实例来说明最清楚不过了，比如在 Drupal 中定义了一个 Content Type 为 news（新闻），新闻关联了若干自定义字段，包括：

- 新闻分类 field_category 一个新闻可以对应多个目录
- 新闻格式 field_format

等等，其中新闻分类 field_category 又关联到一个已有的类别（taxonomy）。这个例子是一个非常常见的多对多结构。

而此时我们希望开放一个 API，形如：

```plain
GET http://avnpc.com/drupalapi/news?tid=<tid>
```

可以通过更改 URL 中的`tid`参数，将该分类下的新闻全部筛选出来。操作方法如下：

首先还是要确认一下 Services Views 模块是否已经安装

__1. 添加一个新的视图（View） Home › Administration › Structure › Views › Add new view__

- View name : news
- Show : Content
- of type : 新闻
- 勾选 Create a block 
- Continue & edit

__2. 在 View 中创建 Service，设置路径__

- Add -> 选择 Services

可以看到 View 中新建了一个 Services 标签页

- Services settings -> Path -> 设置为 news

__3. 设置外部可以更改的参数__

- Filter criteria -> Add
- 勾选 Content: Has taxonomy term -> Apply
- 选择要关联的 Taxonomy Vocabulary ， Selection type 建议为 Dropdown -> Apply & continue
- 勾选  Expose this filter to visitors, to allow them to change it 
- 在 UI 最下方新出现的 More 中， 设置 Filter identifier 为`tid`
- Apply

__4. 保存并公开 Service API__

保存这个 View，在 Home › Administration › Structure > drupalapi 中已经可以看到刚才新建的 news。勾选将其公开出去。

然后可以尝试访问一下

```plain
GET http://avnpc.com/drupalapi/news?tid=7351
```


### 解决 Services Views 产生的缓存不更新 BUG

经过测试，通过上面方法公开的 Views API 是无法根据 URL 参数来设置不同缓存的，有人已经给出了[解决 Views Cache 问题的 Patch](https://drupal.org/node/1055616#comment-7034686)但是似乎还没有正式合并入发布分支，只能自己先打上补丁了。

下载补丁后放在 views 模块下，运行

```plain
patch -pNUM views_1055616_81_cache.patch
```

即可


## Drupal RESTFul API 创建/更新资源

通过上面的方法，已经可以将 Drupal 的所有资源以 RESTFul API 的形式公布给第三方用户，从而可以实现数据的导出或与第三方的交互。而如果要向 Drupal 中导入数据，则需要访问 API 的写入接口。

下面为了更好说明，URL 请求部分采用[Requests](https://github.com/rmccue/Requests)来实现。

### 权限认证

如果没有安装其他第三方认证模块，Drupal RESTFul 的权限认证默认是通过 Session 来进行的：

``` php
Requests::post('http://avnpc.com/apiv1/user/login.json', array(
    'Content-type' => 'application/json'
), json_encode(array('username' => 'yourname', 'password' => '123456')));
```

这样可以取得一个`session_name`和`sessid`，之后的写入请求需要将这两部分用`=`拼接后附加在 header 的`Cookie`中即可。

但是由于旧版本的 Drupal Services 还存在[CSRF](https://drupal.org/node/1890222)安全漏洞，最新的 Drupal Services 中还加入了`X-CSRF-Token`的限制。需要携带登陆后的 Session 信息去换取一个 CSRF Token：

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


### 如何通过 Drupal RESTFul API 创建附带自定义字段的资源

Drupal 的最大优势就是可以灵活的为 Node 绑定任意自定义字段，那么在通过 RESTFul API 创建资源时，也避免不了会有自定义字段的写入。

还是以上文中的 news 资源为例，创建一个 news 资源的代码如下：

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

可能最重要的部分就是让 POST 提交的数据结构保持与自定义 node 字段的结构完全一致。那么如何获得一个完整的 Node 创建时所需要的数据结构？一个非常简单的办法是找到 node 模块下的 node.module。对`node_submit($node)`的`$node`设置断点，然后进入后台创建一个 node，就可以看到 node 创建所需要的完整数据结构了。


## 参考

- [Services 3 - POST node.create with custom fields](https://drupal.org/node/1354202)



---
published: true
date: '2013-07-01 16:17:13'
tags:
  - Drupal
author: AlloVince
title: 实战 Drupal 之 Drupal6 到 Drupal7 的免停机平滑升级
---

近期的工作中，Drupal 的升级与数据迁移占了很大比重。按照[Drupal 官方的升级指南](https://drupal.org/documentation/upgrade/6/7)，建议停站并进入维护模式，但是由于手头的数据量非常大，并且伴随有数据导入等过程，直接在服务器上操作未免存在风险，因此通过一台过渡服务器的方式实现了更加稳妥的[Drupal6 到 Drupal7 的平滑升级](http://avnpc.com/pages/upgrade-drupal6-to-drupal7-without-maintenance-mode)，特记录于此。

## 准备工作

基本的思路是提前准备一台机器作为过渡，一般用自己的电脑就可以了。将旧的服务器数据导出到本地，在本地升级后再部署到新的服务器，最终切换 DNS 就可以实现平滑过渡。

那么为了方便说明，将用到的服务器分别命名如下：

- RemoteOld 即正在在线运行的老站点（Drupal 6）
- Local   过渡服务器
- RemoteNew 升级后的在线运行的新站点（Drupal 7）


## 导出数据并建立本地镜像

首先导出 RemoteOld 数据库，并备份 RemoteOld 的图片文件，在本地搭建一个一模一样的镜像 Local：

```plain
[root@RemoteOld ~] # mysqldump --skip-lock-tables --default-character-set=utf8 -u root -p drupal > /root/RemoteOld.sql
[root@RemoteOld ~] # cd ~ && tar -zcvf RemoteOld.tar.gz  RemoteOld.sql
[root@RemoteOld ~] # cd /var/www/drupal/ && tar -zcvf drupal.tar.gz  ckuploadimg upload sites/default/files/ sites/default/upfile
[root@Local ~] # cd ~ && wget http://yourwebsite/RemoteOld.tar.gz
[root@Local ~] # tar -xvf RemoteOld.tar.gz
[root@Local ~] # mysql -u root -p --default-character-set=utf8
mysql> CREATE DATABASE drupal;
mysql> use drupal;
mysql> source /root/RemoteOld.sql
```

## 数据升级

*1*. 在本地建立一个 RemoteOld 的镜像，数据库链接到上面导入后的 drupal 数据库，并以管理员身份登入 Drupal 后台

*2*. Administer › Site building ›  Modules 停用所有非核心模块，同时记录下所有用到的非核心模块

*3*. Administer › Site buildin › Themes 中设置 Garland 为默认主题

*4*. 清空数据库中所有以 cache_ 和 search_ 开头的表以及 sessions 表

``` sql
TRUNCATE `cache_block`;
TRUNCATE `cache_browscap`;
TRUNCATE `cache_content`;
TRUNCATE `cache_filter`;
TRUNCATE `cache_form`;
TRUNCATE `cache_google_appliance`;
TRUNCATE `cache_menu`;
TRUNCATE `cache_page`;
TRUNCATE `cache_update`;
TRUNCATE `cache_views`;
TRUNCATE `cache_views_data`;
TRUNCATE `search_dataset`;
TRUNCATE `search_index`;
TRUNCATE `search_node_links`;
TRUNCATE `search_total`;
TRUNCATE `sessions`;
```

*5*. 准备一个全新的 Drupal 7

编辑配置文件 sites/default/settings.php，数据库连接到 drupal

方便操作建议绑定一个测试域名到这个 Drupal 7 站点，假设为`drupal.local`

将 sites/default/settings.php 中 `$update_free_access` 设置为 true

``` php
$update_free_access = true;
```

*6*. 解决数据冲突

在升级中发现旧版本的 files 表存在同名文件导致数据无法升级，因此需要先删除重复数据

``` sql
delete files from files  where fid not in (
    select fid from (select fid from files group by filename) as temp_table
);
```

*7*. 运行升级程序

```plain
http://drupal.local/update.php
```

*8*. 重置权限

升级后发现一些权限没有继承，建议使用将 role_permission 表用 Drupal 7 原始数据覆盖。

*9*. 按照上面的记录，下载 Drupal 7 所对应的非核心模块，重新进入后台开启所有用到的模块。

如果模块特别多，依赖比较复杂时，建议不要一次全部启用，可以每次 3～5 个分批启用，每次启用后如果有数据库的升级，后台会提示。按提示再次运行升级程序即可


## 导出 Local 数据并将其导入 RemoteNew

几乎和开始的操作一样，不再赘述。

最后将 DNS 解析切换至新服务器 IP，就完成了一次 Drupal 无停机的平滑升级。


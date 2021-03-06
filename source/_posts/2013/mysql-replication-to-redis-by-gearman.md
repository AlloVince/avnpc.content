---
published: true
date: '2013-07-30 17:30:41'
tags:
  - php
  - Redis
  - Gearman
  - MySQL UDF
author: AlloVince
title: 通过 Gearman 实现 MySQL 到 Redis 的数据复制
---

对于变化频率非常快的数据来说，如果还选择传统的静态缓存方式（File System 等）展示数据，可能在缓存的存取上会有很大的开销，并不能很好的满足需要，而 Redis 这样基于内存的 NoSQL 数据库，就非常适合担任实时数据的容器。

但是往往我们又有数据可靠性的需求，采用 MySQL 作为数据存储，不会因为内存问题而引起数据丢失，同时也可以利用关系数据库的特性实现很多功能。

所以就会很自然的想到是否可以采用 MySQL 作为数据存储引擎，Redis 则作为 Cache。而这种需求目前还没有看到有特别成熟的解决方案或工具，因此本文将尝试采用[Gearman+PHP+MySQL UDF 的组合异步实现 MySQL 到 Redis 的数据复制](http://avnpc.com/pages/mysql-replication-to-redis-by-gearman)。

## MySQL 到 Redis 数据复制方案

无论 MySQL 还是 Redis，自身都带有数据同步的机制，像比较常用的[MySQL 的 Master/Slave 模式](http://dev.mysql.com/doc/refman/5.5/en/replication.html)，就是由 Slave 端分析 Master 的 binlog 来实现的，这样的数据复制其实还是一个异步过程，只不过当服务器都在同一内网时，异步的延迟几乎可以忽略。

那么理论上我们也可以用同样方式，分析 MySQL 的 binlog 文件并将数据插入 Redis。但是这需要对 binlog 文件以及 MySQL 有非常深入的理解，同时由于[binlog 存在 Statement/Row/Mixedlevel 多种形式](http://dev.mysql.com/doc/refman/5.5/en/binary-log-setting.html)，分析 binlog 实现同步的工作量是非常大的。

因此这里选择了一种开发成本更加低廉的方式，借用已经比较成熟的 MySQL UDF，将 MySQL 数据首先放入 Gearman 中，然后通过一个自己编写的 PHP Gearman Worker，将数据同步到 Redis。比分析 binlog 的方式增加了不少流程，但是实现成本更低，更容易操作。


### Gearman 的安装与使用

[Gearman](http://gearman.org/)是一个支持分布式的任务分发框架。设计简洁，获得了非常广泛的支持。一个典型的 Gearman 应用包括以下这些部分：

![Gearman构架](http://gearman.org/images/gearman_stack.png)

- Gearman Job Server：Gearman 核心程序，需要编译安装并以守护进程形式运行在后台
- Gearman Client：可以理解为任务的收件员，比如我要在后台执行一个发送邮件的任务，可以在程序中调用一个 Gearman Client 并传入邮件的信息，然后就可以将执行结果立即展示给用户，而任务本身会慢慢在后台运行。
- Gearman Worker：任务的真正执行者，一般需要自己编写具体逻辑并通过守护进程方式运行，Gearman Worker 接收到 Gearman Client 传递的任务内容后，会按顺序处理。

以前曾经介绍过类似的[后台任务处理项目 Resque](http://avnpc.com/pages/run-background-task-by-php-resque)。两者的设计其实非常接近，简单可以类比为：

- Gearman Job Server：对应 Resque 的 Redis 部分
- Gearman Client：对应 Resque 的 Queue 操作
- Gearman Worker：对应 Resque 的 Worker 和 Job

这里之所以选择 Gearman 而不是 Resque 是因为 Gearman 提供了比较好用的 MySQL UDF，工作量更小。

#### 安装 Gearman 及 PHP Gearman 扩展

以下均以 Ubuntu12.04 为例。

```plain
apt-get install gearman gearman-server libgearman-dev
```

检查 Gearman 的运行状况：

```plain
/etc/init.d/gearman-job-server status
* gearmand is running
```

说明 Gearman 已经安装成功。

[PHP 的 Gearman 扩展](http://pecl.php.net/package/gearman)可以通过 pecl 直接安装

```plain
pecl install gearman
echo "extension=gearman.so" > /etc/php5/conf.d/gearman.ini
service php5-fpm restart
```

但是实测发现 ubuntu 默认安装的 gearman 版本过低，直接运行`pecl install gearman`会报错

> configure: error: libgearman version 1.1.0 or later required

因此 Gearman + PHP 扩展建议通过编译方式安装，这里为了简单说明，选择安装旧版本扩展：

```plain
pecl install gearman-1.0.3
```

#### Gearman + PHP 实例

为了更容易理解后文 Gearman 的运行流程，这里不妨从一个最简单的 Gearman 实例来说明，比如我们要进行一个文件处理的操作，首先编写一个 Gearman Client 并命名为 client.php：

``` php
<?php
$client = new GearmanClient();
$client->addServer();
$client->doBackground('writeLog', 'Log content');
echo '文件已经在后台操作';
```

运行这个文件，相当于模拟用户请求一个 Web 页面后，将处理结束的信息返回用户：

```plain
php client.php
```

查看一下 Gearman 的状况：

```plain
(echo status ; sleep 0.1) | netcat 127.0.0.1 4730
```

可以看到输出为

```plain
writeLog        1       0       0
.
```

说明我们已经在 Gearman 中建立了一个名为 writeLog 的任务，并且有 1 个任务在队列等待中。

而上面的 4 列分别代表[当前的 Gearman 的运行状态](http://search.cpan.org/~dormando/Gearman-Server/lib/Gearman/Server/Client.pm)：

1. 任务名称
2. 在等待队列中的任务
3. 正在运行的任务
4. 正在运行的 Worker 进程

可以使用 watch 进行实时监控：

```plain
watch -n 1 "(echo status; sleep 0.1) | nc 127.0.0.1 4730"
```

然后我们需要编写一个 Gearman Worker 命名为 worker.php：

``` php
<?php
$worker = new GearmanWorker();
$worker->addServer();
$worker->addFunction('writeLog', 'writeLog');
while($worker->work());

function writeLog($job)
{
        $log = $job->workload();
        file_put_contents(__DIR__ . '/gearman.log', $log . "\n", FILE_APPEND | LOCK_EX);
}
```


Worker 使用一个 while 死循环实现守护进程，运行

```plain
php worker.php
```

可以看到 Gearman 状态变为：

```plain
writeLog        0       0       1
```

同时查看同目录下 gearman.log，内容应为从 Client 传入的值`Log content`。

### 通过 MySQL UDF + Trigger 同步数据到 Gearman

MySQL 要实现与外部程序互通的最好方式还是通过[MySQL UDF（MySQL user defined functions）](http://www.mysqludf.org/)来实现。为了让 MySQL 能将数据传入 Gearman，这里使用了[lib_mysqludf_json](https://github.com/mysqludf/lib_mysqludf_json#readme)和[gearman-mysql-udf](https://launchpad.net/gearman-mysql-udf)的组合。

#### 安装 lib_mysqludf_json

使用 lib_mysqludf_json 的原因是因为 Gearman 只接受字符串作为入口参数，可以通过 lib_mysqludf_json 将 MySQL 中的数据编码为 JSON 字符串

``` shell
apt-get install libmysqlclient-dev
wget https://github.com/mysqludf/lib_mysqludf_json/archive/master.zip
unzip master.zip
cd lib_mysqludf_json-master/
rm lib_mysqludf_json.so
gcc $(mysql_config --cflags) -shared -fPIC -o lib_mysqludf_json.so lib_mysqludf_json.c
```

可以看到重新编译生成了 lib_mysqludf_json.so 文件，此时需要查看 MySQL 的插件安装路径：

```plain
mysql -u root -pPASSWORD --execute="show variables like '%plugin%';"
+---------------+------------------------+
| Variable_name | Value                  |
+---------------+------------------------+
| plugin_dir    | /usr/lib/mysql/plugin/ |
+---------------+------------------------+
```

然后将 lib_mysqludf_json.so 文件复制到对应位置：

```plain
cp lib_mysqludf_json.so /usr/lib/mysql/plugin/
```

最后登入 MySQL 运行语句注册 UDF 函数：

``` sql
CREATE FUNCTION json_object RETURNS STRING SONAME 'lib_mysqludf_json.so';
```

#### 安装 gearman-mysql-udf

方法几乎一样：

``` shell
apt-get install libgearman-dev
wget https://launchpad.net/gearman-mysql-udf/trunk/0.6/+download/gearman-mysql-udf-0.6.tar.gz
tar -xzf gearman-mysql-udf-0.6.tar.gz
cd gearman-mysql-udf-0.6
./configure --with-mysql=/usr/bin/mysql_config --libdir=/usr/lib/mysql/plugin/
make && make install
```

登入 MySQL 运行语句注册 UDF 函数：

``` sql
CREATE FUNCTION gman_do_background RETURNS STRING SONAME 'libgearman_mysql_udf.so';
CREATE FUNCTION gman_servers_set RETURNS STRING SONAME 'libgearman_mysql_udf.so';
```

最后指定 Gearman 服务器的信息：

``` sql
SELECT gman_servers_set('127.0.0.1:4730');
```

#### 通过 MySQL 触发器实现数据同步

最终同步哪些数据，同步的条件，还是需要根据实际情况决定，比如我希望将数据表`data`的数据在每次更新时同步，那么编写 Trigger 如下：

``` sql
DELIMITER $$
CREATE TRIGGER datatoredis AFTER UPDATE ON data
  FOR EACH ROW BEGIN
    SET @ret=gman_do_background('syncToRedis', json_object(NEW.id as `id`, NEW.volume as `volume`));
  END$$
DELIMITER ;
```

尝试在数据库中更新一条数据查看 Gearman 是否生效。


### Gearman PHP Worker 将 MySQL 数据异步复制到 Redis

Redis 作为时下当热的 NoSQL 缓存解决方案无需过多介绍，其安装及使用也非常简单：

``` shell
apt-get install redis-server
pecl install redis
echo "extension=redis.so" > /etc/php5/conf.d/redis.ini
```

然后编写一个 Gearman Worker：redis_worker.php 

``` php
#!/usr/bin/env php
<?
$worker = new GearmanWorker();
$worker->addServer();
$worker->addFunction('syncToRedis', 'syncToRedis');

$redis = new Redis();
$redis->connect('127.0.0.1', 6379);

while($worker->work());
function syncToRedis($job)
{
        global $redis;
        $workString = $job->workload();
        $work = json_decode($workString);
        if(!isset($work->id)){
                return false;
        }
        $redis->set($work->id, $workString);
}
```

最后需要将 Worker 在后台运行：

```plain
nohup php redis_worker.php &
```

通过这种方式将 MySQL 数据复制到 Redis，经测试单 Worker 基本可以瞬时完成。

## 注意点

在实际操作中发现，Gearman UDF 在每次 MySQL 服务重启后会丢失已经设置的服务器信息。因为时间有限没有深入的调查原因，而用了曲线救国的解决方法，让 MySQL 在每次服务启动时自动运行一次设置语句：

```plain
vi /var/lib/mysql/init_file.sql
```

加入

``` sql
SELECT gman_servers_set('127.0.0.1:4730');
```

然后在/etc/mysql/my.cnf 的`[mysqld]`小节下加入

```plain
init-file=/var/lib/mysql/init_file.sql
```

然后重启服务。


## 参考

- [MySQL replication to Redis cache server via Ruby, Gearman, triggers, and MySQL UDF (Ubuntu version)](http://ericlondon.com/posts/254-mysql-replication-to-redis-cache-server-via-ruby-gearman-triggers-and-mysql-udf-ubuntu-version)
- [管理 Gearman](http://huoding.com/2012/10/30/196)





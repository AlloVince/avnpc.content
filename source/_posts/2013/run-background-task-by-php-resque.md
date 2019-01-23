---
published: true
date: '2013-01-24 23:48:40'
tags:
  - php
  - resque
  - queue
  - Redis
  - php-resque
author: AlloVince
title: PHP 的轻量消息队列 php-resque 使用说明
---

## 消息队列处理后台任务带来的问题

项目中经常会有后台运行任务的需求，比如发送邮件时，因为要连接邮件服务器，往往需要 5-10 秒甚至更长时间，如果能先给用户一个成功的提示信息，然后在后台慢慢处理发送邮件的操作，显然会有更好的用户体验。

为了实现类似的需求，Web 项目中一般的实现方法是使用消息队列(Message Queue)，比如[MemcacheQ](http://memcachedb.org/memcacheq/)，[RabbitMQ](http://www.rabbitmq.com/)等等，都是很著名的产品。

消息队列说白了就是一个最简单的先进先出队列，队列的一个成员就是一段文本。正是因为消息队列实在太简单了，当拿着消息队列时，反而有点无从下手的感觉，因为这仅仅一个发送邮件的任务，就会引申出很多问题：

1. 消息队列只能存储字符串类型的数据，如何将一个发送邮件这样的“任务”，转换为消息队列中的一个“消息”?
2. 消息队列只负责数据的存放与进出，本身不能执行任何程序，那么我们要如何从消息队列中一个一个取出数据，再将这些数据转化回任务并执行。
3. 我们无法预知消息队列何时会有数据产生，所以我们的任务执行程序还需要具备监控消息队列的能力，也就是一个常驻后台的守护进程。
4. 一般的 Web 应用 PHP 都以 cgi 方式运行，无法常驻内存。我们知道 php 还有[cli 模式](http://www.php-cli.com/)，那么守护进程是否能以 php cli 来实现，效率如何？
5. 当守护进程运行时，Web 应用能否与后台守护进程交互，实现开启/杀死进程的功能以及获得进程的运行状态？

## Resque 对后台任务的设计与角色划分

对以上这些问题，目前为止我能找到的最好答案，并不是来自 php，而是来自 Ruby 的项目[Resque](https://github.com/defunkt/resque)，正是由于 Resque 清晰简单的解决了后台任务带来的一系列问题，Resque 的设计也被 Clone 到 Python、php、NodeJs 等语言：比如 Python 下的[pyres](https://github.com/binarydud/pyres)以及 PHP 下的[php-resque](https://github.com/chrisboulton/php-resque)等等，这里有[各种语言版本的 Resque 实现](https://github.com/defunkt/resque/wiki/Alternate-Implementations)，而在本篇日志里，我们当然要以 PHP 版本为例来说明[如何用 php-resque 运行一个后台任务](http://avnpc.com/pages/run-background-task-by-php-resque)，可能一些细节方面会与 Ruby 版有出入，但是本文中以 php 版为准。

Resque 是这样解决这些问题的：

### 后台任务的角色划分

其实从上面的问题已经可以看出，只靠一个消息队列是无法解决所有问题的，需要新的角色介入。在 Resque 中，一个后台任务被抽象为由三种角色共同完成：

- Job | 任务 ： 一个 Job 就是一个需要在后台完成的任务，比如本文举例的发送邮件，就可以抽象为一个 Job。在 Resque 中一个 Job 就是一个 Class。
- Queue | 队列 ： 也就是上文的消息队列，在 Resque 中，队列则是由[Redis](http://redis.io/)实现的。Resque 还提供了一个简单的队列管理器，可以实现将 Job 插入/取出队列等功能。
- Worker | 执行者 ： 负责从队列中取出 Job 并执行，可以以守护进程的方式运行在后台。

那么基于这个划分，一个后台任务在 Resque 下的基本流程是这样的：

1. 将一个后台任务编写为一个独立的 Class，这个 Class 就是一个 Job。
2. 在需要使用后台程序的地方，系统将 Job Class 的名称以及所需参数放入队列。
3. 以命令行方式开启一个 Worker，并通过参数指定 Worker 所需要处理的队列。
4. Worker 作为守护进程运行，并且定时检查队列。
5. 当队列中有 Job 时，Worker 取出 Job 并运行，即实例化 Job Class 并执行 Class 中的方法。

至此就可以完整的运行完一个后台任务。

在 Resque 中，还有一个很重要的设计：一个 Worker，可以处理一个队列，也可以处理很多个队列，并且可以通过增加 Worker 的进程/线程数来加快队列的执行速度。

### php-resque 的安装

需要提前说明的是，由于涉及到进程的开辟与管理，php-resque 使用了 php 的[PCNTL 函数](http://php.net/manual/zh/ref.pcntl.php)，所以只能在 Linux 下运行，并且需要 php 编译 PCNTL 函数。如果希望用 Windows 做同样的工作，那么可以去找找 Resque 的其他语言版本，php 在 Windows 下非常不适合做后台任务。

以 Ubuntu12.04LTS 为例，Ubuntu 用 apt 安装的 php 已经默认编译了 PCNTL 函数，无需任何配置，以下指令均为 root 帐号

### 安装 Redis

``` shell
apt-get install redis-server
```

###安装 Composer

``` shell
apt-get install curl
cd /usr/local/bin
curl -s http://getcomposer.org/installer | php
chmod a+x composer.phar
alias composer='/usr/local/bin/composer.phar'
```

### 使用 Composer 安装 php-resque

假设 web 目录在/opt/htdocs

``` shell
apt-get install git git-core
cd /opt/htdocs
git clone git://github.com/chrisboulton/php-resque.git
cd php-resque
composer install
```

## php-resque 的使用

### 编写一个 Worker

其实 php-resque 已经给出了简单的例子， demo/job.php 文件就是一个最简单的 Job：

``` php
class PHP_Job
{
    public function perform()
    {
		sleep(120);
		fwrite(STDOUT, 'Hello!');
	}
}
```

这个 Job 就是在 120 秒后向 STDOUT 输出字符 Hello!

在 Resque 的设计中，一个 Job 必须存在一个 perform 方法，Worker 则会自动运行这个方法。

### 将 Job 插入队列

php-resque 也给出了最简单的插入队列实现 demo/queue.php：

``` php
if(empty($argv[1])) {
    die('Specify the name of a job to add. e.g, php queue.php PHP_Job');
}

require __DIR__ . '/init.php';
date_default_timezone_set('GMT');
Resque::setBackend('127.0.0.1:6379');

$args = array(
	'time' => time(),
	'array' => array(
		'test' => 'test',
	),
);

$jobId = Resque::enqueue('default', $argv[1], $args, true);
echo "Queued job ".$jobId."\n\n";
```

在这个例子中，queue.php 需要以 cli 方式运行，将 cli 接收到的第一个参数作为 Job 名称，插入名为'default'的队列，同时向屏幕输出刚才插入队列的 Job Id。在终端输入：

```plain
php demo/queue.php PHP_Job
```

结果可以看到屏幕上输出：

```plain
Queued job b1f01038e5e833d24b46271a0e31f6d6
```

即 Job 已经添加成功。注意这里的 Job 名称与我们编写的 Job Class 名称保持一致：PHP_Job

### 查看 Job 运行情况

php-resque 同样提供了查看 Job 运行状态的例子，直接运行：

```plain
php demo/check_status.php b1f01038e5e833d24b46271a0e31f6d6
```

可以看到输出为：

```plain
Tracking status of b1f01038e5e833d24b46271a0e31f6d6. Press [break] to stop.
Status of b1f01038e5e833d24b46271a0e31f6d6 is: 1
```

我们刚才创建的 Job 状态为 1。在 Resque 中，一个 Job 有以下 4 种状态：

- Resque_Job_Status::STATUS_WAITING = 1; (等待)
- Resque_Job_Status::STATUS_RUNNING = 2; (正在执行)
- Resque_Job_Status::STATUS_FAILED = 3;  (失败)
- Resque_Job_Status::STATUS_COMPLETE = 4; (结束)

因为没有 Worker 运行，所以刚才创建的 Job 还是等待状态。

###运行 Worker

这次我们直接编写 demo/resque.php：

``` php
<?php
date_default_timezone_set('GMT');
require 'job.php';
require '../bin/resque';
```

可以看到一个 Worker 至少需要两部分：

1. 可以直接包含 Job 类文件，也可以使用 php 的自动加载机制，指定好 Job Class 所在路径并能实现自动加载
2. 包含 Resque 的默认 Worker： bin/resque

在终端中运行：

```plain
QUEUE=default php demo/resque.php
```

前面的 QUEUE 部分是设置环境变量，我们指定当前的 Worker 只负责处理 default 队列。也可以使用

```plain
QUEUE=* php demo/resque.php
```

来处理所有队列。

运行后输出为

```plain
#!/usr/bin/env php
*** Starting worker
```

用 ps 指令检查一下：

```plain
ps aux | grep resque
```

可以看到有一个 php 的守护进程已经在运行了

```plain
1000      4607  0.0  0.1  74816 11612 pts/3    S+   14:52   0:00 php demo/resque.php
```

再使用之前的检查 Job 指令

```plain
php demo/check_status.php b1f01038e5e833d24b46271a0e31f6d6
```

2 分钟后可以看到

```plain
Status of b1f01038e5e833d24b46271a0e31f6d6 is: 4
```

任务已经运行完毕，同时屏幕上应该可以看到输出的 Hello!

至此我们已经成功的完成了一个最简单的 Resque 实例的全部演示，更复杂的情况以及遗留的问题会在下一次的日志中说明。
    


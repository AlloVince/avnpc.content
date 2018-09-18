---
published: true
date: '2014-12-26 23:40:08'
tags:
  - php
  - xhprof
author: AlloVince
title: 使用xhprof进行线上PHP性能追踪及分析
---

之前一直使用[基于Xdebug进行PHP的性能分析](http://avnpc.com/pages/how-to-debug-under-zf2)，对于本地开发环境来说是够用了，但如果是线上环境的话，xdebug消耗较大，配置也不够灵活，因此线上环境建议使用[xhprof进行PHP性能追踪及分析](http://avnpc.com/pages/profiler-php-performance-online-by-xhprof)。


## xhprof的安装与简易用法

[xhprof](https://github.com/phacility/xhprof)是Facebook开源的轻量级PHP性能分析工具，Linux环境下可以通过pecl直接安装，比如在Ubuntu下仅需3行指令

``` shell
pecl install xhprof-beta
echo "extension=xhprof.so" > /etc/php5/fpm/conf.d/xhprof.ini
service php5-fpm restart
```

之后可以通过`phpinfo()`检查扩展是否已经加载。

具体如何使用呢，[xhprof项目](https://github.com/phacility/xhprof)中已经提供了示例以及简易的UI，下载xhprof项目到web服务器，假设可以通过`http://localhost/xhprof/`访问，那么访问`http://localhost/xhprof/examples/sample.php`可以看到一些输出，并且提示通过访问`http://<xhprof-ui-address>/index.php?run=XXX&source=xhprof_foo`查看结果。接下来访问`http://localhost/xhprof/xhprof_html/`就可以看到已经保存的结果，列出了所有函数的调用以及所消耗的时间。

分析一下示例代码`sample.php`，关键部分只有2行：

``` php
//开启xhprof并开始记录
xhprof_enable();
//运行一些函数
foo();
//停止记录并取到结果
$xhprof_data = xhprof_disable();
```

`$xhprof_data`中记录了程序单步运行过程中所有的函数调用时间及CPU内存消耗等，具体记录哪些指标可以通过[`xhprof_enable`的入口参数](http://php.net/manual/zh/xhprof.constants.php)控制，之后的处理已经与xhprof扩展无关，大致是编写了一个存储类`XHProfRuns_Default`，将`$xhprof_data`序列化并保存到某个目录，可以通过`XHProfRuns_Default(__DIR__)`将结果输出到当前目录，如果不指定则会读取php.ini配置文件中的`xhprof.output_dir`，仍然没有指定则会输出到`/tmp`。

`xhprof_html/index.php`将记录的结果整理并可视化，默认的UI里列出了：

- funciton name ： 函数名
- calls: 调用次数
- Incl. Wall Time (microsec)： 函数运行时间（包括子函数）
- IWall%：函数运行时间（包括子函数）占比
- Excl. Wall Time(microsec)：函数运行时间（不包括子函数）
- EWall%：函数运行时间（不包括子函数）

每一项应该不难理解，以项目自带的`sample.php`为例，示例中编写了一个`main()`函数，`main()`函数中调用`foo()`、`bar()`等一些子函数进行了一点字符处理。整个程序运行过程中，`main()`函数只运行了一次，并且由于`main()`函数中包括了所有的逻辑，所以`main()`函数的IWall%占比为100%，但是由于`main()`函数的功能都是由子函数实现的，因此`main()`函数的EWall%只有0.3%，而`foo()`函数完成了主要的工作，EWall%有98.1%。因此在分析更大型的程序时，往往需要根据这几项指标分别排序，从不同的角度审视性能消耗。

在`xhprof_html/index.php`中还可以看到`[View Full Callgraph]`链接，点击后可以绘制出一张可视化的性能分析图，如果点击后报错的话，可能是缺少依赖`graphviz`，ubuntu可以通过apt安装

``` shell
apt-get install graphviz
```


##更好的注入方式

了解了上面这些，其实就已经可以将xhprof整合到任何我们已有的项目中去了。目前大部分MVC框架都有唯一的入口文件，只需要在入口文件的开始处注入xhprof的逻辑

``` php
//开启xhprof
xhprof_enable(XHPROF_FLAGS_MEMORY | XHPROF_FLAGS_CPU);
//在程序结束后收集数据
register_shutdown_function(function() {
    $xhprof_data        = xhprof_disable();

    //让数据收集程序在后台运行
    if (function_exists('fastcgi_finish_request')) {
        fastcgi_finish_request();
    }

    //保存xhprof数据
    ...
});
```

但是这样免不了要修改项目的源代码，其实php本身就提供了更好的注入方式，比如将上述逻辑保存为`/opt/inject.php`，然后修改php fpm配置文件

```
vi /etc/php5/fpm/php.ini
```
    vi /etc/php5/fpm/php.ini

修改`auto_prepend_file`配置

```
auto_prepend_file = /opt/inject.php
```

这样所有的php-fpm请求的php文件前都会自动注入`/opt/inject.php`文件

如果使用Nginx的话，还可以通过Nginx的配置文件设置，这样侵入性更小，并且可以实现基于站点的注入。

```
fastcgi_param PHP_VALUE "auto_prepend_file=/opt/inject.php";
```

    
##更好的分析工具：xhprof.io还是xhpgui

注入代码后我们还需要实现保存xhprof数据以及展示数据的UI，听起来似乎又是一大堆工作，有现成的轮子可以用吗？

经过搜索和比较，貌似比较好的选择有[xhprof.io](https://github.com/gajus/xhprof.io)以及[xhpgui](https://github.com/perftools/xhgui)。

两个项目做得事情差不多，都提供了xhprof数据保存功能以及一套索引展示数据的UI，下面是一些比较

xhprof.io

- ✗ 年久失修
- ✗ 保存xhprof数据到MySQL
- ✓ 支持域名、URI等多个维度的数据索引
- ✓ 函数调用记录完整，内核级别函数都能显示
- ✗ 无法针对个别URI开启
- ✗ 注入被分割成两个文件，如果程序被强制中断时xhprof数据将无法收集

xhgui

- ✓ 保存xhprof数据到MongoDB
- ✗ 不支持域名索引
- ✗ 函数调用记录不完整，部分内核级别函数（如扩展内）无法显示
- ✓ 有配置文件可以控制开启条件
- ✓ 注入只有一个文件
- ✓ 狂拽酷炫的基于D3.js的调用关系动态图

可以看到其实两个项目都不够完善，相对而言xhgui不支持域名索引对于线上调试来说是无法忍受的，因此我最后的选择是使用xhprof.io，但是自己进行了微量的调整，修改后的[xhprof.io修正版](https://github.com/EvaEngine/xhprof.io)支持：

- ✓ 增加开启开关配置，可以针对个别URI开启
- ✓ 注入文件合并为一个

## xhprof.io修正版安装与使用

安装及配置方法如下，假设web服务器根目录为`/opt/htdocs`

``` shell
cd /opt/htdocs
git clone https://github.com/EvaEngine/xhprof.io.git
cd xhprof.io/
composer install
cp xhprof/includes/config.inc.sample.php xhprof/includes/config.inc.php
vi xhprof/includes/config.inc.php
```

在MySQL中建立xhprof.io数据库，假设数据库名为`xhprof`，然后导入xhprof/setup/database.sql

配置文件`config.inc.php`中需要调整

- `'url_base' => 'http://localhost/xhprof.io/',` 这是xhprof.io界面所在路径
- `'pdo' => new PDO('mysql:dbname=xhprof;host=localhost;charset=utf8', 'root', 'password'),` 根据MySQL实际情况调整配置
- `enable` 这是一个匿名函数，当匿名函数返回true时启用xhprof数据收集

通过配置`enable`项，就可以实现线上调试的需求，比如

**始终开启xhprof**

``` php
'enable' => function() {
    return true;
}
```

**1/100概率随机开启xhprof**

``` php
'enable' => function() {
    return rand(0, 100) === 1;
}
```

**网页携带参数debug=1时开启xhprof**

``` php
'enable' => function() {
    return !empty($_GET['debug']);
}
```

**网页URL为特定路径时开启**

``` php
'enable' => function() {
    return strpos($_SERVER['REQUEST_URI'], '/testurl') === 0;
}
```

最后按上文所述，在要配置的项目中包含`xhprof.io/inc/inject.php`即可。

线上环境操作时务必要胆大心细，如果没有结果尤其注意需要检查**xhprof扩展是否安装**。

### 附录：xhpgui的安装方法

``` shell
apt-get install mongodb php5-mongo php5-mcrypt
cp /etc/php5/mods-available/mcrypt.ini /etc/php5/fpm/conf.d/
cp /etc/php5/mods-available/mcrypt.ini /etc/php5/cli/conf.d/
cd /opt/htdocs
git clone https://github.com/perftools/xhgui.git
cd xhgui
composer install
cp config/config.default.php config/config.php
chown www-data.www-data -R cache
```

编辑Nginx配置文件加入

```
fastcgi_param PHP_VALUE "auto_prepend_file=/opt/htdocs/xhgui/external/header.php";
```

收集数据过多时可以清空mongodb

```
mongo
use xhprof;
db.dropDatabase();
```

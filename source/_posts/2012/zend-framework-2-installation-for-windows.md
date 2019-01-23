---
published: true
date: '2012-10-06 17:16:05'
tags:
  - Apache
  - ZF2
  - Zend Framework 2
  - XAMPP
  - Windows
author: AlloVince
title: ZF2 入门：Windows 环境下从零开始 Zend Framework 2.0 (ZF2)环境搭建
---

[Zend Framework 2.0 (ZF2)正式发布](http://avnpc.com/pages/zend-framework-2-0-released)之后不少朋友都进行了尝试，可能由于 ZF2 涉及到的新特性比较多，有朋友希望能有一篇从零开始 Zend Framework 2.0 (ZF2)的教程，于是就有了本篇日志。

以下将记录在 Windows 环境下，[从零开始搭建系统并运行一个 ZF2 项目的全过程](http://avnpc.com/pages/zend-framework-2-installation-for-windows/)以及所有需要注意的细节。为了简化整个过程，我没有加入 Git 的安装，改为下载代码，安装环境也使用了傻瓜化的 XAMPP。

一、Apache + MySQL + PHP5.4 环境搭建
==================================

其实 PHP5.4 已经集成了 Web 服务器，但是为了更加简化，我在这里选择了集成安装包[XAMPP](http://www.apachefriends.org/zh_cn/xampp.html)来搭建环境。

安装 [xampp-win32-1.8.0-VC9-installer](http://www.apachefriends.org/zh_cn/xampp-windows.html#1787) 至 D:\

启动 XAMPP Control Panel，最新的 XAMPP 已经集成了 Apache 2.4.2, MySQL 5.5.27, PHP 5.4.7 等最新版本的组件，点击 start 按钮启动 Apache 与 MySQL 服务。启动成功即可在浏览器中访问http://localhost/。

然后进入 http://localhost/security/index.php， 为 mysql 设置一个密码并重新启动 MySQL 服务。

如果想使用 Nginx 配置 ZF2 环境，可以参考[让 XAMPP 使用 Nginx 作为 Web 服务器](http://avnpc.com/pages/add-nginx-to-xampp)


二、部署代码
============

下载实例程序 [ZendSkeletonApplication](https://github.com/zendframework/ZendSkeletonApplication)

解压至 D:\xampp\htdocs 并重命名为 ZendSkeletonApplication

下载[Zend Framework 2.0 最新代码](http://framework.zend.com/downloads/latest)，解压至

```plain
    D:\xampp\htdocs\ZendSkeletonApplication\vendor\ZF2
```

确认一下现在我们的文件结构应该是

```plain
    file://D:\xampp\htdocs
	|   +---ZendSkeletonApplication
	|   |   +---config
	|   |   +---data
	|   |   +---module
	|   |   |   +---Application
	|   |   |       +---config
	|   |   |       +---language
	|   |   |       +---src
	|   |   |       |   +---Application
	|   |   |       |       +---Controller
	|   |   |       +---view
	|   |   |           +---application
	|   |   |           |   +---index
	|   |   |           +---error
	|   |   |           +---layout
	|   |   +---public
	|   |   |   +---css
	|   |   |   +---images
	|   |   |   +---js
	|   |   +---vendor
	|   |       +---ZF2
	|   |           +---bin
	|   |           +---library
	|   |           |   +---Zend
	|   |           +---vendor
```

三、绑定域名
============

编辑 C:\Windows\System32\drivers\etc\hosts

添加任意开发环境用域名：

```plain
    127.0.0.1       zf2.local
    127.0.0.1       www.zf2.local
```

可以访问 http://zf2.local 测试是否已经生效。

然后编辑 Apache 配置文件 D:\xampp\apache\conf\extra\httpd-vhosts.conf 为

```plain
    <VirtualHost *:80>
	ServerName localhost
	DocumentRoot "D:\xampp\htdocs"
	</VirtualHost>

	<VirtualHost *:80>
	ServerName zf2.local
	ServerAlias www.zf2.local
	DocumentRoot "D:\xampp\htdocs\ZendSkeletonApplication\public"
	</VirtualHost>
```


记得重启 Apache 服务。在浏览器中重新访问 http://zf2.local 就可以打开 ZendSkeletonApplication 测试程序了。

至此，一个最基本的 ZF2 项目连同环境已经搭建完毕，可以去修改 zf2 的项目代码去开始一个自己的项目了。



进阶设置
==================

下面的设置不是必须的，但是建议更改以便获得更多功能。

###修改 php.ini 设置

编辑 D:\xampp\php\php.ini

调整错误信息级别

```plain
    error_reporting = E_ALL & ~E_STRICT
```

打开短标签支持，方便 ZF2 模板编写:

```plain
    short_open_tag = On
```

加载 php 多语言插件（Internationalization Functions）支持，这是 ZF2 的 I18N 必须的

```plain
    extension=php_intl.dll
```

开启 Openssl 支持，Oauth 等一些组件必须

```plain
    extension=php_openssl.dll
```

##开启 xDebug

参考日志[Zend2(ZF2)的 Debug 及性能分析方法](http://avnpc.com/pages/how-to-debug-under-zf2)


###安装 Imagick 库 For PHP5.4

下载[Imagick for windows 版本](http://image_magick.veidrodis.com/image_magick/binaries/)，这里请选择 ImageMagick-6.7.7-4-Q16-windows-dll.exe，下载后安装在 C:\ImageMagick。安装过程中注意勾选“Add application directory to your system path”。

安装完毕后最好重启一次计算机，否则可能会有 CORE_RL_wand_.dll 丢失的报警。

下载[php_imagick.dll for php5.4](http://www.peewit.fr/imagick/)，由于 XAMPP 编译的 php 是线程安全（Thread Safe）的，我们需要下载对应的 Thread Safe 版本。

将 php_imagick.dll 放于

```plain
    D:\xampp\php\ext
```

然后编辑 php.ini，加入

```plain
    extension=php_imagick.dll
```
    
最后重启 apache，查看 phpinfo()，安装成功的话会出现相应的 imagick 段落。





---
slug: zend-framework-2-installation-for-windows
date: '2012-10-06 17:16:05'
title: ZF2入门：Windows环境下从零开始Zend Framework 2.0 (ZF2)环境搭建
id: 159
tags:
  - Apache
  - ZF2
  - Zend Framework 2
  - XAMPP
  - Windows
---

[Zend Framework 2.0 (ZF2)正式发布](http://avnpc.com/pages/zend-framework-2-0-released)之后不少朋友都进行了尝试，可能由于ZF2涉及到的新特性比较多，有朋友希望能有一篇从零开始Zend Framework 2.0 (ZF2)的教程，于是就有了本篇日志。

以下将记录在Windows环境下，[从零开始搭建系统并运行一个ZF2项目的全过程](http://avnpc.com/pages/zend-framework-2-installation-for-windows/)以及所有需要注意的细节。为了简化整个过程，我没有加入Git的安装，改为下载代码，安装环境也使用了傻瓜化的XAMPP。

一、Apache + MySQL + PHP5.4环境搭建
==================================

其实PHP5.4已经集成了Web服务器，但是为了更加简化，我在这里选择了集成安装包[XAMPP](http://www.apachefriends.org/zh_cn/xampp.html)来搭建环境。

安装 [xampp-win32-1.8.0-VC9-installer](http://www.apachefriends.org/zh_cn/xampp-windows.html#1787) 至 D:\

启动XAMPP Control Panel，最新的XAMPP已经集成了Apache 2.4.2, MySQL 5.5.27, PHP 5.4.7等最新版本的组件，点击start按钮启动Apache与MySQL服务。启动成功即可在浏览器中访问http://localhost/。

然后进入 http://localhost/security/index.php， 为mysql设置一个密码并重新启动MySQL服务。

如果想使用Nginx配置ZF2环境，可以参考[让XAMPP使用Nginx作为Web服务器](http://avnpc.com/pages/add-nginx-to-xampp)


二、部署代码
============

下载实例程序 [ZendSkeletonApplication](https://github.com/zendframework/ZendSkeletonApplication)

解压至D:\xampp\htdocs并重命名为ZendSkeletonApplication

下载[Zend Framework 2.0最新代码](http://framework.zend.com/downloads/latest)，解压至

    D:\xampp\htdocs\ZendSkeletonApplication\vendor\ZF2

确认一下现在我们的文件结构应该是

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

三、绑定域名
============

编辑 C:\Windows\System32\drivers\etc\hosts

添加任意开发环境用域名：

    127.0.0.1       zf2.local
    127.0.0.1       www.zf2.local

可以访问 http://zf2.local 测试是否已经生效。

然后编辑Apache配置文件 D:\xampp\apache\conf\extra\httpd-vhosts.conf 为

    <VirtualHost *:80>
	ServerName localhost
	DocumentRoot "D:\xampp\htdocs"
	</VirtualHost>

	<VirtualHost *:80>
	ServerName zf2.local
	ServerAlias www.zf2.local
	DocumentRoot "D:\xampp\htdocs\ZendSkeletonApplication\public"
	</VirtualHost>


记得重启Apache服务。在浏览器中重新访问 http://zf2.local 就可以打开ZendSkeletonApplication测试程序了。

至此，一个最基本的ZF2项目连同环境已经搭建完毕，可以去修改zf2的项目代码去开始一个自己的项目了。



进阶设置
==================

下面的设置不是必须的，但是建议更改以便获得更多功能。

###修改php.ini设置

编辑 D:\xampp\php\php.ini

调整错误信息级别

    error_reporting = E_ALL & ~E_STRICT

打开短标签支持，方便ZF2模板编写:

    short_open_tag = On

加载php多语言插件（Internationalization Functions）支持，这是ZF2的I18N必须的

    extension=php_intl.dll

开启Openssl支持，Oauth等一些组件必须

    extension=php_openssl.dll

##开启xDebug

参考日志[Zend2(ZF2)的Debug及性能分析方法](http://avnpc.com/pages/how-to-debug-under-zf2)


###安装Imagick库 For PHP5.4

下载[Imagick for windows版本](http://image_magick.veidrodis.com/image_magick/binaries/)，这里请选择ImageMagick-6.7.7-4-Q16-windows-dll.exe，下载后安装在C:\ImageMagick。安装过程中注意勾选“Add application directory to your system path”。

安装完毕后最好重启一次计算机，否则可能会有CORE_RL_wand_.dll丢失的报警。

下载[php_imagick.dll for php5.4](http://www.peewit.fr/imagick/)，由于XAMPP编译的php是线程安全（Thread Safe）的，我们需要下载对应的Thread Safe版本。

将php_imagick.dll放于

    D:\xampp\php\ext

然后编辑php.ini，加入

    extension=php_imagick.dll
    
最后重启apache，查看phpinfo()，安装成功的话会出现相应的imagick段落。





---
slug: zend-framework-2-installation-for-linux-ubuntu
published: true
date: '2012-10-10 10:46:16'
tags:
  - ZF2
  - Zend Framework 2
  - nginx
  - Git
  - Ubuntu
author: AlloVince
title: ZF2入门：Ubuntu/Linux环境下从零开始Zend Framework 2.0 (ZF2)环境搭建
---

紧接上一篇[ZF2入门：Windows环境下从零开始Zend Framework 2.0 (ZF2)环境搭建](http://avnpc.com/pages/zend-framework-2-installation-for-windows)，本次是[Linux/Ubuntu环境下从零开始搭建系统并运行一个ZF2项目的全过程](http://avnpc.com/pages/zend-framework-2-installation-for-windows)。

写日志的Linux用的是Ubuntu12.04 LTS 32bit版本，为了简化整个过程，没有直接编译，全部采用了apt-get安装软件包。另外本次为了更全的覆盖可能的情况，服务器采用了Nginx，代码部署直接采用Git，Windows下同样可以借鉴本篇的配置。

日志直接以root身份运行，普通用户记得在所有指令前加sudo


一、Nginx + MySQL + PHP5.3环境搭建
=================================

Ubuntu12.04 LTS通过apt安装的默认php版本是5.3.10，php5.4需要编译安装，鉴于php5.3.10运行ZF2已经足够，所以本次就不再考虑php5.4的情况。

    apt-get update
    apt-get upgrade
    apt-get install mysql-server mysql-client nginx php5-fpm php5-mysql php5-curl php5-gd php5-intl php-pear php5-imagick php5-mcrypt php5-memcached git git-core

安装完毕后运行

    service nginx start

然后访问http://localhost应该就可以看到Nginx的Hello World了。

二、部署代码
============

个人习惯将www目录放在/opt/htdocs，请根据环境目录不同对应调整下面的路径及配置：

    cd /opt
    mkdir htdocs
    cd htdocs
    git clone git://github.com/zendframework/ZendSkeletonApplication.git zf2
    cd zf2
    git submodule update --init

短短几行指令，代码就已经部署好了。

三、绑定域名
============

    vi /etc/hosts

同样可以添加任意开发环境用域名：

    127.0.0.1       zf2.local
    127.0.0.1       www.zf2.local

可以访问 http://zf2.local 测试是否已经生效。

编辑Nginx配置文件

    vi /etc/nginx/sites-enabled/default

修改为

    server {
	        listen   80 default;
	        index index.html index.htm;
	        server_name localhost;

	        location / {
	                root /opt/htdocs;
	                index index.php index.html index.htm;
	                try_files $uri $uri/ /index.html;
	        }

	        location ~ \.php {
	                include fastcgi_params;
	                fastcgi_pass   127.0.0.1:9000;
	                fastcgi_index  index.php;
	                fastcgi_param  SCRIPT_FILENAME  /opt/htdocs$fastcgi_script_name;
	        }

	}


	server {
	        listen   80;
	        server_name  zf2.local www.zf2.local;
	        location / {
	                root  /opt/htdocs/zf2/public;
	                index index.php index.html index.htm;
	                if (!-e $request_filename){
	                        rewrite ^/(.*)$ /index.php last;
	                }
	        }
	        location ~ \.php$ {
	                include fastcgi_params;
	                fastcgi_pass   127.0.0.1:9000;
	                fastcgi_index  index.php;
	                fastcgi_param  SCRIPT_FILENAME  /opt/htdocs/zf2/public/$fastcgi_script_name;
	        }
	}

上半段是将Nginx的www根目录更改为/opt/htdocs。下半段是将zf2.local测试域名绑定到/opt/htdocs/zf2/public

重启Nginx服务

    service nginx restart

在浏览器中重新访问 http://zf2.local 就可以打开ZendSkeletonApplication测试程序了。

至此，一个Ubuntu下最基本的ZF2项目连同环境已经搭建完毕，可以去修改zf2的项目代码去开始一个自己的项目了。其他Linux发行版可以类推，CentOS同样可以很方便的用Yum安装。

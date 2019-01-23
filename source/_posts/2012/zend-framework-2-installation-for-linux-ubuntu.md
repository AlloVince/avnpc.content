---
published: true
date: '2012-10-10 10:46:16'
tags:
  - ZF2
  - Zend Framework 2
  - nginx
  - Git
  - Ubuntu
author: AlloVince
title: ZF2 入门：Ubuntu/Linux 环境下从零开始 Zend Framework 2.0 (ZF2)环境搭建
---

紧接上一篇[ZF2 入门：Windows 环境下从零开始 Zend Framework 2.0 (ZF2)环境搭建](http://avnpc.com/pages/zend-framework-2-installation-for-windows)，本次是[Linux/Ubuntu 环境下从零开始搭建系统并运行一个 ZF2 项目的全过程](http://avnpc.com/pages/zend-framework-2-installation-for-windows)。

写日志的 Linux 用的是 Ubuntu12.04 LTS 32bit 版本，为了简化整个过程，没有直接编译，全部采用了 apt-get 安装软件包。另外本次为了更全的覆盖可能的情况，服务器采用了 Nginx，代码部署直接采用 Git，Windows 下同样可以借鉴本篇的配置。

日志直接以 root 身份运行，普通用户记得在所有指令前加 sudo


一、Nginx + MySQL + PHP5.3 环境搭建
=================================

Ubuntu12.04 LTS 通过 apt 安装的默认 php 版本是 5.3.10，php5.4 需要编译安装，鉴于 php5.3.10 运行 ZF2 已经足够，所以本次就不再考虑 php5.4 的情况。

```plain
    apt-get update
    apt-get upgrade
    apt-get install mysql-server mysql-client nginx php5-fpm php5-mysql php5-curl php5-gd php5-intl php-pear php5-imagick php5-mcrypt php5-memcached git git-core
```

安装完毕后运行

```plain
    service nginx start
```

然后访问http://localhost 应该就可以看到 Nginx 的 Hello World 了。

二、部署代码
============

个人习惯将 www 目录放在/opt/htdocs，请根据环境目录不同对应调整下面的路径及配置：

```plain
    cd /opt
    mkdir htdocs
    cd htdocs
    git clone git://github.com/zendframework/ZendSkeletonApplication.git zf2
    cd zf2
    git submodule update --init
```

短短几行指令，代码就已经部署好了。

三、绑定域名
============

```plain
    vi /etc/hosts
```

同样可以添加任意开发环境用域名：

```plain
    127.0.0.1       zf2.local
    127.0.0.1       www.zf2.local
```

可以访问 http://zf2.local 测试是否已经生效。

编辑 Nginx 配置文件

```plain
    vi /etc/nginx/sites-enabled/default
```

修改为

```plain
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
```

上半段是将 Nginx 的 www 根目录更改为/opt/htdocs。下半段是将 zf2.local 测试域名绑定到/opt/htdocs/zf2/public

重启 Nginx 服务

```plain
    service nginx restart
```

在浏览器中重新访问 http://zf2.local 就可以打开 ZendSkeletonApplication 测试程序了。

至此，一个 Ubuntu 下最基本的 ZF2 项目连同环境已经搭建完毕，可以去修改 zf2 的项目代码去开始一个自己的项目了。其他 Linux 发行版可以类推，CentOS 同样可以很方便的用 Yum 安装。

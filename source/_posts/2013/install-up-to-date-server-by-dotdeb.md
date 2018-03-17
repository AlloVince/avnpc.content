---
slug: install-up-to-date-server-by-dotdeb
date: '2013-08-19 14:13:00'
title: Ubuntu12.04使用Dotdeb安装PHP5.4 / Nginx1.4/Redis2.6等新版本
id: 188
tags:
  - php
  - Ubuntu
  - Debian
  - Apt-get
---

众所周知，Ubuntu使用apt-get默认安装的软件版本都偏低，目前Ubuntu12.04安装的PHP版本为PHP Version 5.3.10-1ubuntu3.7，Nginx、Redis等常用软件版本也都非常保守。而这对于个人开发而言，要尝试新版本特性还需要编译安装解决依赖问题，实在不够方便。

[Dotdeb](http://www.dotdeb.org/)就为Debian系提供了一个非常好的高版本更新源，由个人维护，但是更新非常快，使用[Detdeb在Ubuntu12.04上就可以默认安装PHP5.4 / Nginx1.4 / Redis2.6等更新的版本](http://avnpc.com/pages/install-up-to-date-server-by-dotdeb)。

用法也非常简单：

```
vi /etc/apt/sources.list
```

在更新源中加入

```
#dotdeb
deb http://packages.dotdeb.org squeeze all
deb-src http://packages.dotdeb.org squeeze all
deb http://packages.dotdeb.org squeeze-php54 all
deb-src http://packages.dotdeb.org squeeze-php54 all
```

然后添加密钥

```
wget http://www.dotdeb.org/dotdeb.gpg
cat dotdeb.gpg | sudo apt-key add -
```

就可以通过`apt-get update`更新了。


不过在实际使用中发现在安装`php5-mysql`等一些PHP扩展时报错依赖版本不够，要求`libmysqlclient16 (>= 5.1.21-1)`，此时可能需要手动安装

```
wget https://launchpad.net/ubuntu/+archive/primary/+files/libzip1_0.9.3-1_amd64.deb
sudo dpkg -i libzip1_0.9.3-1_amd64.deb

wget http://launchpadlibrarian.net/94808408/libmysqlclient16_5.1.58-1ubuntu5_amd64.deb
sudo dpkg -i libmysqlclient16_5.1.58-1ubuntu5_amd64.deb
```

目前使用Dotdeb在Ubuntu12.04安装的PHP版本为PHP Version 5.4.17-1~dotdeb.0





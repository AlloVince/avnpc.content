---
slug: build-php-develop-env-by-docker
date: '2015-06-30 17:19:40'
title: Docker在PHP项目开发环境中的应用
---

环境部署是所有团队都必须面对的问题，随着系统越来越大，依赖的服务也越来越多，比如我们目前的一个项目就会用到：

- Web服务器：Nginx
- Web程序：PHP + Node
- 数据库：MySQL
- 搜索引擎：ElasticSearch
- 队列服务：Gearman
- 缓存服务：Redis + Memcache
- 前端构建工具：npm + bower + gulp
- PHP CLI工具：Composer + PHPUnit

因此团队的开发环境部署随之暴露出若干问题：

1. 依赖服务很多，本地搭建一套环境成本越来越高，初级人员很难解决环境部署中的一些问题
2. 服务的版本差异及OS的差异都可能导致线上环境BUG
3. 项目引入新的服务时所有人的环境需要重新配置

对于问题1，可以用[Vagrant](https://www.vagrantup.com/)这样的基于虚拟机的项目来解决，团队成员共享一套开发环境镜像。对于问题2，可以引入类似[PHPBrew](https://github.com/phpbrew/phpbrew)这样的多版本PHP管理工具来解决。但两者都不能很好地解决问题3，因为虚拟机镜像没有版本管理的概念，当多人维护一个镜像时，很容易出现配置遗漏或者冲突，一个很大的镜像传输起来也不方便。

Docker的出现让上面的问题有了更好的解决方案，虽然个人对于Docker大规模应用到生产环境还持谨慎态度，但如果仅仅考虑测试及开发，私以为Docker的容器化理念已经是能真正解决环境部署问题的银弹了。

下面介绍[Docker构建PHP项目开发环境](http://avnpc.com/pages/build-php-develop-env-by-docker)过程中的演进，本文中假设你的操作系统为Linux，已经安装了Docker，并且已经了解[Docker是什么](https://www.docker.com/whatisdocker/)，以及[Docker命令行的基础使用](https://docs.docker.com/userguide/)，如果没有这些背景知识建议先自行了解。


## Hello World

首先还是从一个PHP在Docker容器下的Hello World实例开始。我们准备这样一个PHP文件`index.php`：

```
<?php
echo "PHP in Docker";
```

然后在同目录下创建文本文件并命名为`Dockerfile`，内容为：

```
# 从官方PHP镜像构建
FROM       php

# 将index.php复制到容器内的/var/www目录下
ADD        index.php /var/www/

# 对外暴露8080端口
EXPOSE     8080

# 设置容器默认工作目录为/var/www
WORKDIR    /var/www/

# 容器运行后默认执行的指令
ENTRYPOINT ["php", "-S", "0.0.0.0:8080"]
```

构建这个容器：

```
docker build -t allovince/php-helloworld .
```

运行这个容器

```
docker run -d -p 8080:8080 allovince/php-helloworld
```

查看结果：

```
curl localhost:8080
PHP in Docker
```

这样我们就创建了一个用于演示PHP程序的Docker容器，任何安装过Docker的机器都可以运行这个容器获得同样的结果。而任何有上面的php文件和Dockerfile的人都可以构建出相同的容器，从而完全消除了不同环境，不同版本可能引起的各种问题。


想象一下程序进一步复杂，我们应该如何扩展呢，很直接的想法是继续在容器内安装其他用到的服务，并将所有服务运行起来，那么我们的Dockerfile很可能发展成这个样子：

```
FROM       php
ADD        index.php /var/www/

# 安装更多服务
RUN	       apt-get install -y \
	       mysql-server \
	       nginx \
	       php5-fpm \
	       php5-mysql
	       
# 编写一个启动脚本启动所有服务
ENTRYPOINT ["/opt/bin/php-nginx-mysql-start.sh"]
```

虽然我们通过Docker构建了一个开发环境，但觉不觉得有些似曾相识呢。没错，其实这种做法和制作一个虚拟机镜像是差不多的，这种方式存在几个问题：

- 如果需要验证某个服务的不同版本，比如测试PHP5.3/5.4/5.5/5.6，就必须准备4个镜像，但其实每个镜像只有很小的差异。
- 如果开始新的项目，那么容器内安装的服务会不断膨胀，最终无法弄清楚哪个服务是属于哪个项目的。


## 使用单一进程容器

上面这种将所有服务放在一个容器内的模式有个形象的非官方称呼：Fat Container。与之相对的是将服务分拆到容器的模式。从Docker的设计可以看到，构建镜像的过程中可以指定唯一一个容器启动的指令，因此Docker天然适合一个容器只运行一种服务，而这也是官方更推崇的。

分拆服务遇到的第一个问题就是，我们每一个服务的基础镜像从哪里来？这里有两个选项：

**选项一、 统一从标准的OS镜像扩展**，比如下面分别是Nginx和MySQL镜像

```
FROM ubuntu:14.04
RUN  apt-get update -y && apt-get install -y nginx
```

```
FROM ubuntu:14.04
RUN  apt-get update -y && apt-get install -y mysql
```

这种方式的优点在于所有服务可以有一个统一的基础镜像，对镜像进行扩展和修改时可以使用同样的方式，比如选择了ubuntu，就可以使用`apt-get`指令安装服务。

问题在于大量的服务需要自己维护，特别是有时候需要某个服务的不同版本时，往往需要直接编译源码，调试维护成本都很高。

**选项二、 直接从Docker Hub继承官方镜像**，下面同样是Nginx和MySQL镜像

```
FROM nginx:1.9.0
```

```
FROM mysql:5.6
```

[Docker Hub](https://registry.hub.docker.com/)可以看做是Docker的Github，Docker官方已经准备好了大量[常用服务的镜像](https://registry.hub.docker.com/repos/library/?s=stars)，同时也有非常多第三方提交的镜像。甚至可以基于[Docker-Registry](https://github.com/docker/docker-registry)项目在短时间内自己搭建一个私有的Docker Hub。

基于某个服务的官方镜像去构建镜像，有非常丰富的选择，并且可以以很小的代价切换服务的版本。这种方式的问题在于官方镜像的构建方式多种多样，进行扩展时需要先了解原镜像的`Dockerfile`。

出于让服务搭建更灵活的考虑，我们选择后者构建镜像。

为了分拆服务，现在我们的目录变为如下所示结构：

```
~/Dockerfiles
├── mysql
│   └── Dockerfile
├── nginx
│   ├── Dockerfile
│   ├── nginx.conf
│   └── sites-enabled
│       ├── default.conf
│       └── evaengine.conf
├── php
│   ├── Dockerfile
│   ├── composer.phar
│   ├── php-fpm.conf
│   ├── php.ini
│   ├── redis.tgz
└── redis
    └── Dockerfile
```

即为每个服务创建单独文件夹，并在每个服务文件夹下放一个Dockerfile。

### MySQL容器

MySQL继承自官方的[MySQL5.6镜像](https://registry.hub.docker.com/_/mysql)，Dockerfile仅有一行，无需做任何额外处理，因为普通需求官方都已经在镜像中实现了，因此Dockerfile的内容为：


```
FROM mysql:5.6
```

在项目根目录下运行


	docker build -t eva/mysql ./mysql

会自动下载并构建镜像，这里我们将其命名为`eva/mysql`。

由于容器运行结束时会丢弃所有数据库数据，为了不用每次都要导入数据，我们将采用挂载的方式持久化MySQL数据库，官方镜像默认将数据库存放在`/var/lib/mysql`，同时要求容器运行时必须通过环境变量设置一个管理员密码，因此可以使用以下指令运行容器：

	docker run -p 3306:3306 -v ~/opt/data/mysql:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 -it eva/mysql

通过上面的指令，我们将本地的3306端口绑定到容器的3306端口，将容器内的数据库持久化到本地的`~/opt/data/mysql`，并且为MySQL设置了一个root密码`123456`


### Nginx容器

Nginx目录下提前准备了Nginx配置文件`nginx.conf`以及项目的配置文件`default.conf`等。Dockerfile内容为：

```
FROM nginx:1.9

ADD  nginx.conf      /etc/nginx/nginx.conf
ADD  sites-enabled/*    /etc/nginx/conf.d/
RUN  mkdir /opt/htdocs && mkdir /opt/log && mkdir /opt/log/nginx
RUN  chown -R www-data.www-data /opt/htdocs /opt/log

VOLUME ["/opt"]
```

由于官方的[Nginx1.9](https://registry.hub.docker.com/_/nginx/)是基于Debian Jessie的，因此首先将准备好的配置文件复制到指定位置，替换镜像内的配置，这里按照个人习惯，约定`/opt/htdocs`目录为Web服务器根目录，`/opt/log/nginx`目录为Nginx的Log目录。

同样构建一下镜像

	docker build -t eva/nginx ./nginx

并运行容器

	docker run -p 80:80 -v ~/opt:/opt -it eva/nginx

注意我们将本地的80端口绑定到容器的80端口，并将本地的`~/opt`目录挂载到容器的`/opt`目录，这样就可以将项目源代码放在`~/opt`目录下并通过容器访问了。

### PHP容器

PHP容器是最复杂的一个，因为在实际项目中，我们很可能需要单独安装一些PHP扩展，并用到一些命令行工具，这里我们以Redis扩展以及Composer来举例。首先将项目需要的扩展等文件提前下载到php目录下，这样构建时就可以从本地复制而无需每次通过网络下载，大大加快镜像构建的速度：

	wget https://getcomposer.org/composer.phar -O php/composer.phar
	wget https://pecl.php.net/get/redis-2.2.7.tgz -O php/redis.tgz

php目录下还准备好了php配置文件`php.ini`以及`php-fpm.conf`，基础镜像我们选择的是[PHP 5.6-FPM](https://registry.hub.docker.com/_/php/)，这同样是一个Debian Jessie镜像。官方比较亲切的在镜像内部准备了一个`docker-php-ext-install`指令，可以快速安装如GD、PDO等常用扩展。所有支持的扩展名称可以通过在容器内运行`docker-php-ext-install`获得。

来看一下Dockerfile

```
FROM php:5.6-fpm

ADD php.ini    /usr/local/etc/php/php.ini
ADD php-fpm.conf    /usr/local/etc/php-fpm.conf

COPY redis.tgz /home/redis.tgz
RUN docker-php-ext-install gd \
    && docker-php-ext-install pdo_mysql \
    && pecl install /home/redis.tgz && echo "extension=redis.so" > /usr/local/etc/php/conf.d/redis.ini
ADD composer.phar /usr/local/bin/composer
RUN chmod 755 /usr/local/bin/composer

WORKDIR /opt
RUN usermod -u 1000 www-data

VOLUME ["/opt"]
```

在构建过程中做了这样一些事情：

1. 复制php和php-fpm配置文件到相应目录
2. 复制redis扩展源代码到`/home`
3. 通过`docker-php-ext-install`安装GD和PDO扩展
4. 通过`pecl`安装Redis扩展
5. 复制composer到镜像作为全局指令

按照个人习惯，仍然设置`/opt`目录作为工作目录。

这里有一个细节，在复制tar包文件时，使用的Docker指令是`COPY`而不是`ADD`，这是由于`ADD`指令会[自动解压`tar`文件](https://docs.docker.com/reference/builder/#add)。

现在终于可以构建+运行了：

	docker build -t eva/php ./php
	docker run -p 9000:9000 -v ~/opt:/opt -it eva/php

在大多数情况下，Nginx和PHP所读取的项目源代码都是同一份，因此这里同样挂载本地的`~/opt`目录，并且绑定9000端口。

### PHP-CLI的实现

php容器除了运行php-fpm外，还应该作为项目的php cli使用，这样才能保证php版本、扩展以及配置文件保持一致。

例如在容器内运行Composer，可以通过下面的指令实现：

	docker run -v $(pwd -P):/opt -it eva/php composer install --dev -vvv
	
这样在任意目录下运行这行指令，等于动态将当前目录挂载到容器的默认工作目录并运行，这也是PHP容器指定工作目录为`/opt`的原因。

同理还可以实现phpunit、npm、gulp等命令行工具在容器内运行。

### Redis容器

为了方便演示，Redis仅仅作为缓存使用，没有持久化需求，因此Dockerfile仅有一行

```
FROM redis:3.0
```


## 容器的连接

上面已经将原本在一个容器中运行的服务分拆到多个容器，每个容器只运行单一服务。这样一来容器之间需要能互相通信。Docker容器间通讯的方法有两种，一种是像上文这样将容器端口绑定到一个本地端口，通过端口通讯。另一种则是通过Docker提供的[Linking功能](https://docs.docker.com/userguide/dockerlinks/)，在开发环境下，通过Linking通信更加灵活，也能避免端口占用引起的一些问题，比如可以通过下面的方式将Nginx和PHP链接起来：

```
docker run -p 9000:9000 -v ~/opt:/opt --name php -it eva/php
docker run -p 80:80 -v ~/opt:/opt -it --link php:php eva/nginx
```

在一般的PHP项目中，Nginx需要链接PHP，而PHP又需要链接MySQL，Redis等。为了让容器间互相链接更加容易管理，Docker官方推荐使用[Docker-Compose](https://docs.docker.com/compose/)完成这些操作。

用一行指令完成安装

    pip install -U docker-compose

然后在Docker项目的根目录下准备一个docker-compose.yml文件，内容为：


```
nginx:
    build: ./nginx
    ports:
      - "80:80"
    links:
      - "php"
    volumes:
      - ~/opt:/opt

php:
    build: ./php
    ports:
      - "9000:9000"
    links:
      - "mysql"
      - "redis"
    volumes:
      - ~/opt:/opt

mysql:
    build: ./mysql
    ports:
      - "3306:3306"
    volumes:
      - ~/opt/data/mysql:/var/lib/mysql
    environment:
      MYSQL_ROOT_PASSWORD: 123456

redis:
    build: ./redis
    ports:
      - "6379:6379"

```

然后运行`docker-compose up`，就完成了所有的端口绑定、挂载、链接操作。

## 更复杂的实例

上面是一个标准PHP项目在Docker环境下的演进过程，实际项目中一般会集成更多更复杂的服务，但上述基本步骤仍然可以通用。比如[EvaEngine/Dockerfiles](https://github.com/EvaEngine/Dockerfiles)是为了运行我的开源项目[EvaEngine](http://avnpc.com/pages/eva-engine)准备的基于Docker的开发环境，EvaEngine依赖了队列服务Gearman，缓存服务Memcache、Redis，前端构建工具Gulp、Bower，后端Cli工具Composer、PHPUnit等。具体实现方式可以自行阅读代码。

经过团队实践，原本大概需要1天时间的环境安装，切换到Docker后只需要运行10余条指令，时间也大幅缩短到3小时以内（大部分时间是在等待下载），最重要的是Docker所构建的环境都是100%一致的，不会有人为失误引起的问题。未来我们会进一步将Docker应用到CI以及生产环境中。


本文首发于我在[卧龙阁](http://www.wolonge.com/zhuanlan/detail/117441)的专栏[PHP与创业的那些事儿](http://www.wolonge.com/zhuanlan/user/1002562)，转载请保留。



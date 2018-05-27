---
published: true
date: '2013-01-01 00:03:21'
tags:
  - XAMPP
  - nginx
author: AlloVince
title: 在windows环境下让XAMPP使用Nginx作为Web服务器
---

之前已经介绍过[Windows环境下ZF2环境搭建的方法](http://avnpc.com/pages/zend-framework-2-installation-for-windows)，使用了XAMPP这个一键式安装环境，默认集成的Web服务器是Apache，比较方便进行开发。不过由于产品环境往往部署在Nginx下，有时候我们也希望测试Nginx，那么可以通过非常简单的方法[将Nginx加入XAMPP](http://avnpc.com/pages/add-nginx-to-xampp)，并可以使Nginx与Apache并存。


## Nginx的安装与启动

假设XAMPP的安装与上一篇日志一致，即[xampp](http://www.apachefriends.org/zh_cn/xampp-windows.html) 已安装至 D:\，首先[下载Nginx的Window版本](http://nginx.org/en/download.html)，解压至 D:\xampp\nginx

为了启动Nginx，我们需要允许php以cgi方式执行，为了不影响Apache的配置，在这里推荐的方式是复制一个php.ini文档，复制D:\xampp\php\php.ini至D:\xampp\php\php-cli.ini，php-cli.ini就是Nginx专用的配置。

编辑php-cli.ini，打开其中几项配置：

```
enable_dl = On
cgi.force_redirect = 0
cgi.fix_pathinfo=1
fastcgi.impersonate = 1
cgi.rfc2616_headers = 1
```

然后开启一个cmd运行php-cgi：

```
D:\xampp\php\php-cgi.exe -b 127.0.0.1:9000 -c D:\xampp\php\php-cli.ini
```

在另一个cmd下开启nginx

```
D:\xampp\nginx\nginx
```

注意由于没有规定端口，所以Nginx也会使用默认的80端口，如果已经开启了Apache，这里需要先停止Apache。访问localhost可以看到Nginx是否已经运行。

要停止nginx可以运行

```
taskkill /F /IM nginx.exe > nul
taskkill /F /IM php-cgi.exe > nul
```


### Nginx站点配置

Nginx默认的站点目录是Nginx安装路径下的html文件夹。我们通过编辑

```
D:\xampp\nginx\conf\nginx.conf
```

让Nginx使用Xampp的站点目录，配置如下：

```
location / {
    root   D:/xampp/htdocs;
    index  index index.php index.html index.htm;
    autoindex on;
    autoindex_exact_size on;
    autoindex_localtime on;
}
location ~ \.php {
    root D:/xampp/htdocs/;
    include fastcgi.conf;
    fastcgi_pass 127.0.0.1:9000;
    fastcgi_index  index.php;
    fastcgi_param  SCRIPT_FILENAME    $document_root$fastcgi_script_name;
}
```


autoindex是允许文件列表模式，方便在开发环境下调试，如果是运行环境最好关闭。

由于XAMPP还内置了phpmyadmin，并存放在与Web目录并行的 D:\xampp\phpMyAdmin 路径下，我们需要额外配置一下，加入以下配置以支持XAMPP的phpmyadmin：

```
location /phpmyadmin/ {
    root   D:/xampp/;
    index  index.php index.html index.htm;
}
location ~ /phpmyadmin/.*\.php {
    root D:/xampp/;
    include fastcgi.conf;
    fastcgi_pass 127.0.0.1:9000;
    fastcgi_index  index.php;
    fastcgi_param  SCRIPT_FILENAME    D:/xampp/$fastcgi_script_name;
}
```


每次更新配置文件后，需要让Nginx重新加载配置

```
D:\xampp\nginx\nginx.exe -s reload
```

同理我们可以绑定一个测试域名到任意Web目录，以[上一篇日志的ZF2测试站点ZendSkeletonApplication](http://avnpc.com/pages/zend-framework-2-installation-for-windows)为例，加入如下配置：

```
server {
    listen       80;
    server_name  zf2.local www.zf2.local;
    location / {
        root   D:/xampp/htdocs/ZendSkeletonApplication/public/;
        index  index index.php index.html index.htm;
        if (!-e $request_filename){
            rewrite ^/(.*)$ /index.php last;
        }
    }
    location ~ \.php {
        root D:/xampp/htdocs/ZendSkeletonApplication/public/;
        include fastcgi.conf;
        fastcgi_pass 127.0.0.1:9000;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME    $document_root$fastcgi_script_name;
    }
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   html;
    }
}
```

最后确认一下完整的nginx.conf：

```
worker_processes  1;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    gzip  on;
    server {
        listen       80;
        server_name  localhost;
        location /phpmyadmin/ {
            root   D:/xampp/;
            index  index.php index.html index.htm;
        }
        location ~ /phpmyadmin/.*\.php {
            root D:/xampp/;
            include fastcgi.conf;
            fastcgi_pass 127.0.0.1:9000;
            fastcgi_index  index.php;
            fastcgi_param  SCRIPT_FILENAME    D:/xampp/$fastcgi_script_name;
        }
        location / {
            root   D:/xampp/htdocs;
            index  index index.php index.html index.htm;
            autoindex on;
            autoindex_exact_size on;
            autoindex_localtime on;
        }
        location ~ \.php {
            root D:/xampp/htdocs/;
            include fastcgi.conf;
            fastcgi_pass 127.0.0.1:9000;
            fastcgi_index  index.php;
            fastcgi_param  SCRIPT_FILENAME    $document_root$fastcgi_script_name;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }

    server {
        listen       80;
        server_name  zf2.local www.zf2.local;
        location / {
            root   D:/xampp/htdocs/ZendSkeletonApplication/public/;
            index  index index.php index.html index.htm;
            if (!-e $request_filename){
                rewrite ^/(.*)$ /index.php last;
            }
        }
        location ~ \.php {
            root D:/xampp/htdocs/ZendSkeletonApplication/public/;
            include fastcgi.conf;
            fastcgi_pass 127.0.0.1:9000;
            fastcgi_index  index.php;
            fastcgi_param  SCRIPT_FILENAME    $document_root$fastcgi_script_name;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
}
```

### 让Nginx在后台运行

为了更容易的使用Nginx，可以简单的写两个脚本管理Nginx的启动和停止，这里需要一个小程序[RunHiddenConsole](https://skydrive.live.com/redir?resid=1E48DF64F8BD957!202)的辅助，将RunHiddenConsole.exe放置于D:\xampp\RunHiddenConsole.exe

然后在同目录下创建nginx_start.bat

```
@echo off
RunHiddenConsole.exe D:\xampp\php\php-cgi.exe -b 127.0.0.1:9000 -c D:\xampp\php\php-cli.ini
cd D:\xampp\nginx
start nginx
```

nginx_stop.bat：

```
@echo off
taskkill /F /IM nginx.exe > nul
taskkill /F /IM php-cgi.exe > nul
exit
```




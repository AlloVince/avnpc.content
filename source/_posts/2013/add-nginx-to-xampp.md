---
published: true
date: '2013-01-01 00:03:21'
tags:
  - XAMPP
  - nginx
author: AlloVince
title: 在 windows 环境下让 XAMPP 使用 Nginx 作为 Web 服务器
---

之前已经介绍过[Windows 环境下 ZF2 环境搭建的方法](http://avnpc.com/pages/zend-framework-2-installation-for-windows)，使用了 XAMPP 这个一键式安装环境，默认集成的 Web 服务器是 Apache，比较方便进行开发。不过由于产品环境往往部署在 Nginx 下，有时候我们也希望测试 Nginx，那么可以通过非常简单的方法[将 Nginx 加入 XAMPP](http://avnpc.com/pages/add-nginx-to-xampp)，并可以使 Nginx 与 Apache 并存。


## Nginx 的安装与启动

假设 XAMPP 的安装与上一篇日志一致，即[xampp](http://www.apachefriends.org/zh_cn/xampp-windows.html) 已安装至 D:\，首先[下载 Nginx 的 Window 版本](http://nginx.org/en/download.html)，解压至 D:\xampp\nginx

为了启动 Nginx，我们需要允许 php 以 cgi 方式执行，为了不影响 Apache 的配置，在这里推荐的方式是复制一个 php.ini 文档，复制 D:\xampp\php\php.ini 至 D:\xampp\php\php-cli.ini，php-cli.ini 就是 Nginx 专用的配置。

编辑 php-cli.ini，打开其中几项配置：

```plain
enable_dl = On
cgi.force_redirect = 0
cgi.fix_pathinfo=1
fastcgi.impersonate = 1
cgi.rfc2616_headers = 1
```

然后开启一个 cmd 运行 php-cgi：

```plain
D:\xampp\php\php-cgi.exe -b 127.0.0.1:9000 -c D:\xampp\php\php-cli.ini
```

在另一个 cmd 下开启 nginx

```plain
D:\xampp\nginx\nginx
```

注意由于没有规定端口，所以 Nginx 也会使用默认的 80 端口，如果已经开启了 Apache，这里需要先停止 Apache。访问 localhost 可以看到 Nginx 是否已经运行。

要停止 nginx 可以运行

```plain
taskkill /F /IM nginx.exe > nul
taskkill /F /IM php-cgi.exe > nul
```


### Nginx 站点配置

Nginx 默认的站点目录是 Nginx 安装路径下的 html 文件夹。我们通过编辑

```plain
D:\xampp\nginx\conf\nginx.conf
```

让 Nginx 使用 Xampp 的站点目录，配置如下：

```plain
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


autoindex 是允许文件列表模式，方便在开发环境下调试，如果是运行环境最好关闭。

由于 XAMPP 还内置了 phpmyadmin，并存放在与 Web 目录并行的 D:\xampp\phpMyAdmin 路径下，我们需要额外配置一下，加入以下配置以支持 XAMPP 的 phpmyadmin：

```plain
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


每次更新配置文件后，需要让 Nginx 重新加载配置

```plain
D:\xampp\nginx\nginx.exe -s reload
```

同理我们可以绑定一个测试域名到任意 Web 目录，以[上一篇日志的 ZF2 测试站点 ZendSkeletonApplication](http://avnpc.com/pages/zend-framework-2-installation-for-windows)为例，加入如下配置：

```plain
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

最后确认一下完整的 nginx.conf：

```plain
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

### 让 Nginx 在后台运行

为了更容易的使用 Nginx，可以简单的写两个脚本管理 Nginx 的启动和停止，这里需要一个小程序[RunHiddenConsole](https://skydrive.live.com/redir?resid=1E48DF64F8BD957!202)的辅助，将 RunHiddenConsole.exe 放置于 D:\xampp\RunHiddenConsole.exe

然后在同目录下创建 nginx_start.bat

```plain
@echo off
RunHiddenConsole.exe D:\xampp\php\php-cgi.exe -b 127.0.0.1:9000 -c D:\xampp\php\php-cli.ini
cd D:\xampp\nginx
start nginx
```

nginx_stop.bat：

```plain
@echo off
taskkill /F /IM nginx.exe > nul
taskkill /F /IM php-cgi.exe > nul
exit
```




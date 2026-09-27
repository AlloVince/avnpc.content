---
title: "Django with mod_python环境搭建总结"
date: "2008-07-02 17:01:51"
slug: "Django_with_mod_python"
published: true
author: "AlloVince"
legacy_id: 100
comment_status: "closed"
comments: false
tags:
  - "Apache"
  - "Django"
  - "mod_python"
  - "Python"
  - "YD的程序员葛阁"
---
**目录** [折叠]

1. [安装Django](#toc_37045100)
2. [安装Apache的mod_python模块](#toc_37058100)
3. [创建Django项目](#toc_37070100)
4. [基于Location的Mod_python访问配置](#toc_37076400)
5. [Hello Django](#toc_37096400)
6. [吐槽](#toc_37105500)

[Django](http://www.djangoproject.com/)是经常与Rails相提并论的Python Web框架。但别的姑且不说，Django的环境搭建实在是不够亲切，简单的搭建过程中被雷到多次，特记录如下以供备忘。

#### 安装Django

确保在Django安装之前已经安好了Python。

Django的最新Source可以从SVN获得：[http://code.djangoproject.com/svn/django/trunk/](http://code.djangoproject.com/svn/django/trunk/)

然后执行Django的setup.py

```python
python setup.py install
```

这里建议把python目录下的script文件夹加入环境变量

#### 安装Apache的mod_python模块

虽然Django内置了Web服务器，但官方建议使用Apache的mod_python模块（- -|||那你还写他干什么）

安装好Apache后，从[http://www.modpython.org/](http://www.modpython.org/)获得mod_python并安装（注意下载对应的python和Apache版本）

安装完毕后Apache目录下的modules文件夹内会有mod_python.so，在httpd.conf中写入

```python
LoadModule python_module modules/mod_python.so
```

将mod_python模块加载进来然后重启Apache

#### 创建Django项目

进入到在Apache的web页面目录下运行

```python
django-admin.py startproject newtest
```

至此我们的第一个Django项目便成功创建了，此时需要牢记项目名称『newtest』，因为真正的雷才刚刚开始- -。

#### 基于Location的Mod_python访问配置

同样是修改httpd.conf，下面是来自Django官网的配置例子

```python
<Location "/mysite/">
    SetHandler python-program
    PythonHandler django.core.handlers.modpython
    SetEnv DJANGO_SETTINGS_MODULE mysite.settings
    PythonDebug On
    PythonPath "['/path/to/project'] + sys.path"
</Location>
```

这里有三个参数需要注意：

1. Location "/mysite/"：这个参数是最终在url上反映出来的地址，也就是说目前我们访问Django项目所用的url就是http://localhost/mysite/，此参数可以自由更改，阿猫阿狗都可以。
2. mysite.settings ：这里的mysite必须是Django的项目名，即是刚才所创建的『newtest』，mod_python通过项目名来载入settings.py从而加载整个项目。此处设置不当会出现 ImportError: Could not import settings 'xxx.settings' (Is it on sys.path? Does it have syntax errors?): No module named xxx.settings 错误。
3. ['/path/to/project'] ： 这个参数是最雷的，需要设置为相对于Django项目的父目录，也就是运行django-admin.py startproject时的所在目录。

因此我的Apache的htdocs目录位于D:\xampp\htdocs，最终设置结果为

```python
<Location "/newtest/">
    SetHandler python-program
	PythonPath "sys.path+['D:/xampp/htdocs']"
    PythonHandler django.core.handlers.modpython
    SetEnv DJANGO_SETTINGS_MODULE newtest.settings
    PythonDebug On
</Location>
```

然后重启Apache，访问http://localhost/newtest/，就可以看到亲切的django运行画面了。

#### Hello Django

终于可以开始第一个程序了，在Django项目下新建一个helloworld.py文件，内容为

```python
from django.http import HttpResponse

def index(request):
	return HttpResponse("Hello, Django.")
```

然后编辑urls.py

```python
from django.conf.urls.defaults import *

urlpatterns = patterns('',
	(r'^newtest/index', 'newtest.helloworld.index'),
)
```

最后重启Apache - -|||，注意Django的所有配置更新都是需要重启Apache的……通过http://localhost/newtest/index看看结果吧……

#### 吐槽

总体来说Django的初始配置给我的印象是很不友好的，违反惯性思维的路径设置；每追加一个URL就要重启一次服务简直就是IIS的再现；而舍弃成熟的 Mod_rewrite模块不用，将所有Url统一通过PythonHandler用正则表达式解析后转发，比起强大的mod_rewrite来也未必能提高多少灵活性。或许作者有自己的考量，但作为一个快速开发的框架，Django在易用性上还需要更加琢磨吧。

PS：人生はいいことがあったり、悪いことがあったり、そのたびに喜んだり、悲しんだり、と言いうことだな

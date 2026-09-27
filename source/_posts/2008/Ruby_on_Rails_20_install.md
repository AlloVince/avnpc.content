---
title: "Ruby on Rails 2.0的安装笔记"
date: "2008-12-09 15:27:20"
slug: "Ruby_on_Rails_20_install"
published: true
author: "AlloVince"
legacy_id: 121
comment_status: "closed"
comments: false
tags:
  - "rails"
  - "ruby"
  - "YD的程序员葛阁"
---
**目录** [折叠]

1. [安装Ruby](#toc_29181000)
2. [安装Rails](#toc_29189700)
3. [创建项目](#toc_29196300)
4. [配置数据库](#toc_29207900)
5. [配置mysql](#toc_29223900)
6. [Hello world](#toc_29233100)

很久没碰ROR了，没想到Ruby on Rails 2.0的初始安装会有这么大的变化，特记录下自己的Win32 rail2.0 + mysql的Rails2.0安装过程备忘。

#### 安装Ruby

可以从Rail的[官网](http://rubyforge.org/frs/?group_id=167)选择最新的安定版安装。本文为1.8.6-26 Final Release

#### 安装Rails

安装好ruby之后在命令行模式下执行

```sql
gem install rails
```

会自动检测所需的类库并进行安装

#### 创建项目

命令行模式下进入所要创建项目的文件夹，执行

```ruby
rails test
```

即完成了一个名为test项目的创建，然后进入test目录启动服务器

```ruby
ruby script/server
```

即可以在http://localhost:3000看到欢迎页面

然后再创建controller即可

```ruby
ruby script\generate controller MyTest
```

#### 配置数据库

在2.0中，由于默认数据库更换为sqlite3，如果没有安装sqlite3，访问controller会报错：

This error occurred while loading the following files ...sqlite3...

对此需要重新配置config/database.yml，更简单的方法是在创建项目时加入参数，将默认DB改为mysql即可

```ruby
rails test -d mysql
```

此时再配置yaml文件，形如：

```ruby
development:
  adapter: mysql
  encoding: utf8
  database: test
  pool: 5
  username: root
  password: root
  host: localhost
```

需要特别注意的是yaml文件里:后的一个空格是必须的，如果粗心忽略了空格的话，在服务器的记录里会看到诸如

c:/ruby/lib/ruby/1.8/yaml.rb:133:in \`load':.....

之类的报错，实际上是yaml文件的读取错误。

#### 配置mysql

更换到Mysql也并不是万事大吉了，mysql的支持模块默认是没有安装的，只能看到"MissingSourceFile"的错误。

所以首先要在命令行里执行

```ruby
gem install mysql
```

不过我出现了"ERROR: While generating documentation for mysql-2.7.3-x86-mswin32 ...."的更新失败报错，大概是服务器问题，好在只是文档部分无法更新，可以忽略。

重启服务器，此时又提示缺少libmysql.dll文件，将这个文件从mysql的目录复制到ruby的bin目录下，服务器正常启动。

#### Hello world

最后是亲切的Hello world

```ruby
class myTestController < ApplicationController
	def index
		render:text=> "hello world"
	end
end
```

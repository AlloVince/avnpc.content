---
slug: php-open-source-project-plus-travis-ci
published: true
date: '2013-03-06 21:32:47'
tags:
  - php
  - CI
  - PHPUnit
author: AlloVince
title: PHP开源项目使用Travis CI进行持续集成
---

一个项目如何保证代码质量是开发中非常重要的环节，对于开源项目来说更是如此，因为开源项目要面对的是来自不同水平开发者提交的代码。所以围绕开源做持续集成（Continuous Integration）变得越来越重要，而目前使用最广泛的免费CI工具当数[Travis CI](https://travis-ci.org/)，以我的项目[EvaThumber](http://avnpc.com/pages/evathumber)为例，来介绍一下如何在[PHP开源项目中配合Travis CI进行持续集成](http://avnpc.com/pages/php-open-source-project-plus-travis-ci)

Travis CI能做的最主要工作是自动运行项目的单元测试并生成报告。例如进入[EvaThumber的Travis CI页面](https://travis-ci.org/AlloVince/EvaThumber)，可以看到最新版本的测试情况，默认设置下，每次对项目进行Push时，都会触发Travis CI运行一次测试，测试环境包括PHP5.3与PHP5.4两个主流版本。Travis CI同时提供了一个项目状态图标，可以放置在项目主页告知用户当前的测试情况：

EvaThumber Master分支: [![Build Status](https://secure.travis-ci.org/AlloVince/EvaThumber.png?branch=master)](http://travis-ci.org/AlloVince/EvaThumber)

## PHP项目的目录结构

如何开始一个PHP项目的持续集成，首先从项目的目录结构说起。目前的PHP项目一般都会遵守[PHP-FIG](http://www.php-fig.org/)制订的PSR规范，可以根据实际项目与团队的情况选择规范的严格程度，如果是新项目建议尽可能选择[PSR-2](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-2-coding-style-guide.md)或以上级别。

遵守规范的意义在于你的项目可以非常容易的集成到其他项目中，同时因为有了相同的代码书写约定，别人也更加容易阅读你的代码。所以越是[优秀的PHP项目](http://avnpc.com/pages/best-wheels-for-php)，越能严格的遵守规范，这会让整个PHP社区进入一个良性循环。

那么一个遵循PSR-2规范的PHP项目进行CI，推荐目录结构以及必须的文件如下：

```
src/
    EvaThumber/
        Thumber.php
tests/
    EvaThumberTest/
        ThumberTest.php
    Bootstrap.php
    phpunit.xml.dist
vendor/
.travis.yml
composer.json
LICENSE
```


- `src`目录用于存放项目源代码，文件夹命名必须与命名空间一致，以便可以进行类的自动载入
- `tests` 目录放置单元测试代码，推荐与src目录保持一样的结构，在文件夹与文件名后缀Test以避免重名。
- `tests/Bootstrap.php` 不是必须的，但一般都会存在，主要用于初始化类的自动载入功能
- `tests/phpunit.xml.dist` 是PHPUnit的配置文件，一般来说可以简单配置一下要测试的目录以及Bootstrap.php的位置，详细的可配置项参考[官网说明](http://www.phpunit.de/manual/current/en/appendixes.configuration.html)
- `vendor` 目录用于放置项目依赖的第三方类库。
- `.travis.yml` 是Travis CI的配置文件，在这个文件中可以设置项目语言，测试环境等。可以参考[Travis CI详细设置](http://about.travis-ci.org/docs/user/build-configuration/)
- `composer.json` 是Composer管理第三方类库依赖关系的配置文件，参考[Composer详细文档](http://getcomposer.org/doc/01-basic-usage.md)
- `LICENSE` 内文是当前项目使用的许可证


## 依赖管理并加入启动文件

EvaThumber依赖一些第三方项目，所以最终`composer.json`的内容如下：

``` json
{
    "name": "AlloVince/EvaThumber",
    "description": "EvaThumber",
    "license": "BSD-3-Clause",
    "homepage": "http://avnpc.com/",
    "require": {
        "php": ">=5.3.3",
        "imagine/Imagine": "dev-master"
    },
    "require-dev": {
        "rmccue/Requests": "dev-master@dev",
        "aferrandini/phpqrcode": "dev-master@dev",
        "symfony/process": "dev-master@dev"
    },
    "autoload": {
        "psr-0": {
            "Requests": "library/",
            "PHPQRCode": "lib/"
        }
    }
}
```

这个配置文件中首先定义了EvaThumber自己的名称，然后规定php运行环境必须大于等于5.3.3，并且列出了所有依赖项目的名称，autoload栏目则对PSR-0的项目的自动加载单独进行了处理。最终当EvaThumber项目被使用时，只需要在项目目录下运行

```
composer install
```

即可安装依赖。安装完毕后会生成`vendor/autoload.php`文件，由于单元测试也需要自动加载的支持，我们编写`tests/Bootstrap.php`为：

``` php
<?php
$loader = include __DIR__ . '/../vendor/autoload.php';
$loader->add('EvaThumber', __DIR__ . '/../src');
```


## 配置单元测试

有了单元测试的启动文件，为了在每次运行测试时加载，需要编写`tests/phpunit.xml.dist`为：

``` xml
<phpunit bootstrap="./Bootstrap.php" colors="true">
    <testsuites>
        <testsuite name="EvaThumber Test Suite">
            <directory>./EvaThumberTest</directory>
        </testsuite>
    </testsuites>
    <filter>
        <whitelist>
            <directory>../src/EvaThumber/</directory>
            <exclude>
                <directory>../vendor/</directory>
            </exclude>
        </whitelist>
    </filter>
</phpunit>
```

主要配置了三点：

1. 指定启动文件为同目录下的`Bootstrap.php`
2. 指定单元测试用例目录为同目录下的`EvaThumberTest`
3. 指定测试代码路径，并将`vendor`目录排除，因为vendor目录为第三方依赖，无需测试

配置好之后在tests目录下运行

``` shell
phpunit -v
```

即可

### PHPUnit安装

如果没有安装PHPUnit，ubuntu下可以很简单的用以下指令安装：

``` shell
apt-get install php-pear
pear channel-update pear.php.net
pear upgrade-all
pear channel-discover pear.phpunit.de
pear install -a phpunit/PHPUnit
```

## 配置并运行Travis CI

由上面的配置可以知道，如果想测试EvaThumber，必须要安装第三方依赖，同时进入tests目录运行`phpunit -v`。所以Travis CI的配置文件`.travis.yml`如下：

```
language: php
php:
  - 5.3
  - 5.4
before_script:
  - composer install
  - cd tests
script: phpunit -v
```

Travis CI会自动完成Git Clone的工作，`before_script`很好理解，一一录入在开始测试前需要运行的指令即可。

最后进入[Travis CI主页](https://travis-ci.org/)，用Github帐号直接登录。点击自己的名字后，会将自己的开源项目全部列举出来。选择要进行测试的项目，将右边的开关设为On就会自动开始测试。


## 接收外部的提交

如果其他人Fork了EvaThumber并发起了Pull Request，同样会触发Travis CI自动运行，如果提交进来的代码无法通过单元测试，Travis CI会自动回复Pull Request并显示测试报告，这样就能保证已有代码的API不会被破坏。

而如果是Bug Fix或者新功能的添加，那么可以要求提交者配套对应的单元测试，否则不予以Merge。




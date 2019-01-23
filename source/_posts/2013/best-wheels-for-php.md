---
published: true
date: '2013-02-04 21:08:16'
tags:
  - php
  - 开源
author: AlloVince
title: 那些最好的轮子 - PHP 篇
---

在[关于不要重复造轮子的二三事](http://avnpc.com/pages/howto-find-best-wheel-for-programming)一文中，交代了一些背景和想法。本篇则完全是一些干货，列举一些我用过或者即将会用的 PHP 轮子，基本都符合我对好轮子的定义：[开源、许可证宽松、容易集成的 PHP 项目](http://avnpc.com/pages/best-wheels-for-php)，目有些已经集成在[EvaEngine](http://avnpc.com/pages/eva-engine)里面，希望能帮助别人少走弯路。

日志还会陆续补充更新，同时欢迎推荐补充。


## Databse 数据库 ORM


### [Doctrine 2](http://www.doctrine-project.org/)

- License : MIT
- [Source Code](https://github.com/doctrine/doctrine2)
- Allo 点评：Doctrine 是功能最全最完善的 PHP ORM，社区一直很活跃，对 NoSQL 也非常迅速的作出了跟进与支持。但之所以没有说 Doctrine 是最好的，是因为我对 PHP 究竟有没有必要使用如此庞大的 ORM 还心存疑虑，平心而论 Doctrine 的入门门槛实在有些高，尤其是 DBAL 的提出，更是要把开发者牢牢绑定在 Doctrine 这艘大船上，用与不用，还是要仔细权衡。


### [RedBeanPHP](http://www.redbeanphp.com/)

- License : New BSD
- [Source Code](https://github.com/gabordemooij/redbean)
- Allo 点评：相比起 Doctrine，RedBean 轻巧的简直要飞起来，这两个轮子就是一组最好的比照，是大而全，还是小而精，根据项目选择吧。


## Documents & Testing 文档与测试


### [phpDocumentor 2](http://www.phpdoc.org/)

- License : MIT
- [Source Code](https://github.com/phpDocumentor/phpDocumentor2)
- Allo 点评：老牌 php 文档生成工具。


###[Faker](https://github.com/fzaninotto/Faker)  

- License : MIT
- [Source Code](https://github.com/fzaninotto/Faker)
- Allo 点评：Faker 是一个很神奇的项目，会自动生成拟真的数据，包括用户资料、长文本、IP、日期等等，在网站上线前测试时非常好用。


## Datetime 时间处理


### [Carbon](https://github.com/briannesbitt/Carbon)
- License : MIT
- [Source Code](https://github.com/briannesbitt/Carbon)
- Allo 点评：虽然 PHP5 内置的 Datetime 类已经足够应付一般需求，不过 Carbon 所提供的一些更人性化的处理则更符合实际需求，如果是时间相关的项目应该考虑使用。


## File System 文件系统


### [Gaufrette](https://github.com/KnpLabs/Gaufrette)

- License : MIT
- [Source Code](https://github.com/KnpLabs/Gaufrette)
- Allo 点评：文件系统几乎是所有项目都会遇到的问题，Gaufrette 为常见的文件系统提供了一套统一接口，包括本地文件/FTP/Dropbox/GridFS/Zip/AmazonS3 等等，是大型系统必备的组件。


## Front-end 前端性能


### [Assetic](https://github.com/kriswallsmith/assetic)

- License : MIT
- [Source Code](https://github.com/kriswallsmith/assetic)
- Allo 点评：Assetic 可以说生来就是为了多模块的项目而存在的，有了 Assetic，可以将分散在各模块中的前端文件编译、合并、压缩。可以让开发人员专注于代码的编写而不是前端文件的生成。


### [lessphp](http://leafo.net/lessphp/)

- License : MIT
- [Source Code](https://github.com/leafo/lessphp)
- Allo 点评：LESS 编译器的 php 版本。不过对于复杂的 LESS 项目，比如 bootstrap，编译的结果与 NodeJS 原版还是有差异，只能做为 Assetic 的一个补充。


### [minify](https://github.com/mrclay/minify)

- License : MIT
- [Source Code](https://github.com/mrclay/minify)
- Allo 点评：PHP 版本的 CSS/JS 压缩器。


## HTTP Client HTTP 客户端


### [Requests](http://requests.ryanmccue.info/)

- License : MIT
- [Source Code](https://github.com/rmccue/Requests)
- Allo 点评：Requests 实现的非常灵巧，底层默认没有使用 cURL 而是采用 fsockopen 作为通信手段，非常适合集成在一些小型项目中。


### [Buzz](https://github.com/kriswallsmith/Buzz)

- License : MIT
- [Source Code](https://github.com/kriswallsmith/Buzz)
- Allo 点评：另一个轻量级的 HTTP 客户端实现，文档上不够丰富。独到之处在于内置了事件机制，可以更灵活的集成。


## HTML & Dom


### [HTMLPurifier](http://htmlpurifier.org/)

- License : LGPL v2.1+
- [Source Code](https://github.com/ezyang/htmlpurifier)
- Allo 点评：凡是有 WYSIWYG 功能的项目，XSS 以及恶意的提交都会成为一个头痛的问题。HTMLPurifier 提供了完整的 HTML 校验与纠错，又无需安装[tidy 扩展](http://php.net/manual/en/book.tidy.php)。


### [PHP Simple HTML DOM Parser](http://simplehtmldom.sourceforge.net/)

- License : MIT
- [Source Code](http://sourceforge.net/projects/simplehtmldom/files/)
- Allo 点评：解析 HTML 为 DOM 并且可以使用 jQuery 选择器操作 DOM，如果需要提取 HTML 页面内容而不考虑高性能，那么用 PHP Simple HTML DOM 可以很惬意。


## Image 图形处理

### [Imagine](http://imagine.readthedocs.org/en/latest/)

- License : MIT
- [Source Code](https://github.com/avalanche123/Imagine)
- Allo 点评：Imagine 为几大图形处理库提供了一个统一接口，即后台可以为 Gd、Imagick、Gmagick 的任意一种，而代码保持不变。其实 Pear 也提供过类似的库[Image_Transform](http://pear.php.net/package/Image_Transform)，但是 Imagine 明显更胜一筹。我还基于 Imagine 做了一个可以用 URL 操作图片的项目[EvaThumber](http://avnpc.com/pages/evathumber)，可以更加简单的集成。
- 应用范围：缩略图生成等任何图形相关的功能。


## Log 处理


### [Monolog](https://github.com/Seldaek/monolog)

- License : MIT
- [Source Code](https://github.com/Seldaek/monolog)
- Allo 点评：可以非常简单的规定 Log 格式，并有众多的后端支持。虽然像 Zend Framework 也内置了[Zend\Log](http://framework.zend.com/manual/2.0/en/index.html#zend-log)这样的组件，但是 Monolog 仍然是最全面专业的 Log 处理首选方案
- 应用范围：几乎所有需要线上调试或者收集用户信息的系统


## Markups 标记语言


### [PHP Markdown](https://github.com/michelf/php-markdown)

- License : New BSD License
- [Source Code](https://github.com/michelf/php-markdown)
- Allo 点评：Markdown 在轻量级标记语言中已经俨然有一统天下的趋势，PHP Markdown 应该是目前以 PHP 编写的最好的 Markdown 解析器。当然一般来说使用 Markdown 作为标记语言需要搭配一个 JS 编辑器，比如[PageDown-Bootstrap](http://samwillis.co.uk/pagedown-bootstrap/demo/browser/demo.html)
- 应用范围：任何中长篇的用户数据录入，比如用户评论、Blog 等场景。可以减轻用户录入负担，并且有效的防止 XSS


## Payment Gateways 支付网关


### [Aktive Merchant for PHP](https://github.com/akDeveloper/Aktive-Merchant)

- License : MIT
- [Source Code](https://github.com/akDeveloper/Aktive-Merchant)
- Allo 点评：Ruby 项目[Active Merchant](https://github.com/Shopify/active_merchant)的 php 版本。对 PayPal、Authorize.net 等多家支付网关提供了统一的接口。
- 应用范围：需要支付网关的项目，有国内支付宝等网关支付需求的完全可以贡献代码


### [Omnipay](https://github.com/adrianmacneil/omnipay)

- License : MIT
- [Source Code](https://github.com/adrianmacneil/omnipay)
- Allo 点评：统一接口的支付网关，支持的支付接口更丰富一些。


## Queue 任务队列


### [php-resque](https://github.com/chrisboulton/php-resque)

- License : MIT
- [Source Code](https://github.com/chrisboulton/php-resque)
- Allo 点评：php-resque 是 Ruby 项目 resque 在 php 下的实现。虽然[Gearman](http://gearman.org/)也是一个不错的选择，但是 resque 的构架设计更加简洁清晰，更加符合 KISS 原则。简单用法可以参看[用 PHP 实现守护进程任务后台运行与多线程](http://avnpc.com/pages/run-background-task-by-php-resque)一文
- 应用范围：需要后台任务的系统，比如邮件发送、同步信息等需求。


## Templating 模板引擎


### [Twig](http://twig.sensiolabs.org/)

- License : New BSD License
- [Source Code](https://github.com/fabpot/Twig)
- Allo 点评：如果说对模板引擎的印象还停留在 Smarty 的阶段，那么你真的已经落后于时代了。Twig 是目前关注度最高的 PHP 模板引擎，比 Smarty 提供了更简约和易懂的语法。当然如果项目没有主题切换这样的需求，php 本身就是最好的模板引擎。
- 应用范围：有皮肤、主题切换需求的项目，可以避免 php 模板带来的安全问题



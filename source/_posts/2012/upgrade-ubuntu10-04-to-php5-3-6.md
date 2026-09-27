---
title: "在ubuntu10.04下用apt升级php5.3.2到php5.3.3以上版本"
date: "2012-07-17 22:37:20"
slug: "upgrade-ubuntu10-04-to-php5-3-6"
published: true
author: "AlloVince"
legacy_id: 153
comment_status: "open"
comments: true
gitalk_id: "POST_153"
tags: []
---
[Ubuntu10.04始终没有将php5.3.3及以上版本加入官方的apt库](https://bugs.launchpad.net/lucid-backports/+bug/693303 "https://bugs.launchpad.net/lucid-backports/+bug/693303")中，如果不想编译的话，目前只能通过第三方apt源更新。记录一下测试通过的目前可以使用的源：

```
apt-get install python-software-properties
add-apt-repository ppa:fabianarias/php5
apt-get update
```

三行命令即可。

不过上述指令可能会因为没有公钥报错

```
http://ppa.launchpad.net lucid Release: The following signatures couldn't be verified because the public key is not available: NO_PUBKEY A42227CB8D0DC64F
```

需要用以下导入所需公钥（报错的最后8位）

```
gpg --keyserver keyserver.ubuntu.com --recv 8D0DC64F
gpg --export --armor 8D0DC64F | apt-key add -
```

```
apt-get update
```

重启Web服务就可以看到已经[ubuntu已经升级到php5.3.6](/pages/upgrade-ubuntu10-04-to-php5-3-6 "http://avnpc.com/pages/upgrade-ubuntu10.04-to-php5.3.6")了。

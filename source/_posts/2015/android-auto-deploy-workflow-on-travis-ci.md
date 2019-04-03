---
published: true
date: '2015-12-15 00:47:35'
tags:
  - Android
  - Travis
author: AlloVince
title: 基于 Travis CI 搭建 Android 自动打包发布工作流（支持 Github Release 及 fir.im）
---

@[toc]

最近付费购买了[Travis CI](https://travis-ci.com/)，Travis CI 的收费模式很有意思，不是按项目或者用户，而是按工作进程收费，比如初级版本是$129/月，总共提供 2 个工作进程。在项目不多的情况下，除了用于跑单元测试外，不免想利用的更充分一些，因此抽空搭建了一套[基于 Travis CI 的 Android 自动发布工作流](http://avnpc.com/pages/android-auto-deploy-workflow-on-travis-ci)。

未自动化前安卓开发总是避免不了这样的工作流程：

1. 开发一些新功能，提交代码
2. 完成一部分功能后，打包一个测试版 APK
3. 将测试版 APK 上传到 QQ 群 / 网盘 / [Fir.im](http://fir.im/) / [蒲公英](http://www.pgyer.com/)等
4. 在 QQ 群或发布平台解释当前版本所完成的功能
5. 通知测试人员测试

实现了这套自动化发布后，工作流程被简化成：

1. 开发新功能，提交代码
2. 通过`git tag`对代码打一个内测版的 tag，在 tag 的描述中对写当前完成的功能

Tag 提交后 Travis CI 会自动编译代码，生成 APK 文件并分发到 Github 和 fir.im，Github 和 fir.im 中会保持 Tag 的描述信息，分发完成后会有邮件通知所有参与测试的人员。而作为开发人员，只需要专注于对代码打好一个 Tag 就可以了。

整个流程看似做了不少工作，其实体现在 Travis CI 只有数行指令而已，以下逐一讲解：

### 对安卓项目启用 Travis CI

[Travis CI](http://travis-ci.org/)应该可以算是目前最好用的持续集成服务之一了，如果代码库是基于 Github 的话，可以很简单的开启。由于本文涉及到了很多 Travis CI 的基础概念，建议首先对[Travis CI 的自定义构建](https://docs.travis-ci.com/user/customizing-the-build/)一节有所了解。

很早前在介绍 PHP 项目的持续集成时也写过[如何在 PHP 项目中使用 Travis CI](http://avnpc.com/pages/php-open-source-project-plus-travis-ci)。 对于安卓项目来说步骤几乎一致：

首先准备一个`.travis.yml`文件放在安卓项目根目录下，`.travis.yml`中记录了 Travis CI 所需的基础信息：

``` yaml
language: android

sudo: false

android:
  components:
  - build-tools-23.0.1
  - android-23
  - extra-android-m2repository
  - extra-android-support

script:
  - "./gradlew assembleRelease"
```

无需读文档就可以通过上面的配置大概知道，我们要运行的是一个安卓项目，安卓 SDK 版本为 23，项目所用的 BuildTools 版本为 23.0.1，为编译这个项目我们还引入了一些必须的组件，如[Support Library（extra-android-support）](http://developer.android.com/tools/support-library/index.html)、Android Support Repository（extra-android-m2repository）等。

当 Travis CI 准备好我们所需要的环境后，将自动运行 yml 文件`script`部分所设置的指令，上例中运行的是`./gradlew assembleRelease`，运行成功的话会在项目的主模块下生成`build/outputs/apk/app-release.apk`。

最后进入 Travis CI 主页，使用有项目 Admin 权限的 Github 帐号直接登录。选择要开启 Travis CI 的项目，将右边的开关设为 On 即可。

Travis CI 目前有 2 个网站：如果是开源项目，直接进入[travis-ci.org](http://travis-ci.org/)即可，如果是私有付费项目，则需要进入[travis-ci.com](https://travis-ci.com/)，2 个网站除了域名外所有的界面及操作几乎一模一样。

配置中还有一行`sudo: false`，是为了[开启基于容器的 Travis CI 任务](https://docs.travis-ci.com/user/migrating-from-legacy/)，让编译效率更高。

### 安卓自动化构建的密码和证书安全

安卓项目发布需要证书文件和若干密码，但无论是开源项目还是私有项目，任何时候都不应该将原始证书或密码放入代码库（原则上来讲证书和密码也不应该交于开发人员，而应该只能通过发布服务器进行编译）。Travis CI 为此提供了 2 种解决方案，一种是对敏感信息、密码、证书等进行对称加密，在 CI 构建环境时解密，另一种是将密码等通过 Travis CI 的控制台（即网站）设置为构建时的环境变量。

由于前者会在 Travis 控制台生成一对环境变量，所以我的做法是尽量选择后者，但由于 Travis 控制台无法上传文件，因此涉及到文件加密的部分，则只能选择前者。

说了这么多，首先还是需要先对编译脚本进行改造，如果不考虑安全问题，项目的`build.gradle`文件可能会是这样：

```groovy
android {
    signingConfigs {
        releaseConfig {
            storeFile file("../keys/evandroid.jks")
            storePassword "123456"
            keyAlias "evandroid_alias"
            keyPassword "654321"
        }
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
            signingConfig signingConfigs.releaseConfig
        }
    }
}
```

而我们最终要的效果，还是希望一份编译脚本既可以用于开发环境，也可以在 CI 环境下使用，在 Travis CI 中，可以通过点击项目名称 -> Settings -> Environment Variables 中设置环境变量，比如我们可以针对上面的配置，分别设置`KEYSTORE_PASS`、`ALIAS_NAME`、`ALIAS_PASS`三个环境变量，在 Travis CI 环境下可以通过`System.getenv()`获得这些环境变量。

本地开发环境中，我的做法是将这几个变量加到`gradle.properties`文件中，这样就可以在`build.gradle`内直接使用了。下面是开发环境的`gradle.properties`

```bash
KEYSTORE_PASS=123456
ALIAS_NAME=evandroid_alias
ALIAS_PASS=654321
```

这样一来`build.gradle`就变成了

```groovy
        releaseConfig {
            storeFile file("../keys/evandroid.jks")
            storePassword project.hasProperty("KEYSTORE_PASS") ? KEYSTORE_PASS : System.getenv("KEYSTORE_PASS")
            keyAlias project.hasProperty("ALIAS_NAME") ? ALIAS_NAME : System.getenv("ALIAS_NAME")
            keyPassword project.hasProperty("ALIAS_PASS") ? ALIAS_PASS : System.getenv("ALIAS_PASS")
        }
```

接下来处理证书文件，为了方便文件加密等功能，Travis CI 提供了一个基于 ruby 的 CLI 命令行工具，可以直接使用 gem 安装

```bash
gem install travis
```

安装后进入安卓项目根目录，尝试对证书文件加密：

```bash
travis encrypt-file keys/evandroid.jks --add
```

如果首次运行，travis 会提示需要登录，运行`travis login --org`并输入 Github 用户名密码即可。（付费版则为`travis login --pro`）

`travis encrypt-file`指令会做几件事情：

1. 在 Travis CI 控制台自动生成一对密钥，形如：`encrypted_e41864bb9dab_key`, `encrypted_e41864bb9dab_iv`
2. 基于密钥通过`openssl`对文件进行加密，上例中会项目根目录生成`evandroid.jks.enc`文件
3. 在`.travis.yml`中自动生成 Travis CI 环境下解密文件的配置，上例运行后可以看到`.travis.yml`中多了几行：

```yaml
before_install:
- openssl aes-256-cbc -K $encrypted_e41864bb9dab_key -iv $encrypted_e41864bb9dab_i -in keys/evandroid.jks.enc -out keys/evandroid.jks -d
```

Travis CI 默认在项目根目录下运行，因此注意根据实际需求调整 enc 文件的路径。

最后别忘了在`.gitignore`中忽略`keys/evandroid.jks`以及`gradle.properties`并在代码库中将其删除。

### Travis CI 自动发布安卓 apk 文件到 Github Release

Travis CI 的`script`部分运行成功后，可以通过配置文件进入到发布阶段。下面是一个 Travis CI 发布的示例：

```yaml
deploy:
  provider: releases
  user: "GITHUB USERNAME"
  password: "GITHUB PASSWORD"
  file: app/build/outputs/apk/app-release.apk
  skip_cleanup: true
  on:
    tags: true
```
 
这个例子中配置了这样一些内容：

- `provider`：发布目标为 Github Release，除了 Github 外，Travis CI 还支持发布到 AWS、Google App Engine 等数十种[provider](https://docs.travis-ci.com/user/deployment/#Supported-Providers)
- Github 用户名和密码，因为 Travis CI 要上传 APK 文件，因此需要有 Github 项目的写入权限
- `file`： 发布文件，输入文件路径即可
- `skip_cleanup`: 默认情况下 Travis CI 在完成编译后会清除所有生成的文件，因此需要将`skip_cleanup`设置为 true 来忽略此操作。
- `on`： 发布的时机，这里配置为`tags: true`，即只在有`tag`的情况下才发布。

虽然这样就能完成自动发布，但是直接暴露了 Github 密码是我们更加不能接受的。更好的做法是在 Github -> settings -> Personal access tokens 生成一个只能访问当前项目并只有读取权限的[Github Access Token](https://help.github.com/articles/creating-an-access-token-for-command-line-use/)，并通过 Travis CI 将 Access Token 加密。听起来有点繁琐，好在 Travis CLI 中已经可以通过一行指令做好这一切：

```bash
travis setup release
```

根据提示填写上述配置项目的信息后，Travis CLI 会自动在`.travis.yml`文件中生成好所有的配置项：

```yaml
deploy:
  provider: releases
  api_key:
    secure: XXX
  file: app/build/outputs/apk/app-release.apk
  skip_cleanup: true
  on:
    tags: true
    all_branches: true
```

其中`api_key`下的`secure`就是加密后的 Access Token。

在运行`travis setup release`时有可能遇到

> Invalid scheme format: git@github.com
> for a full error report, run travis report

这样的报错，看起来是 Travis CLI 还不支持通过密钥访问 Github，因此可以将项目的源临时切换为 http 形式，运行成功后再切换回来：

```bash
git remote set-url origin https://github.com/AlloVince/evandroid.git
git remote set-url origin git@github.com:AlloVince/evandroid.git
```

在实际部署过程中，发现发布到 Github Release 比较坑的点是

```bash
git push
git push --tags
```

往往会同时生成 2 个 Travis CI 任务，但是在 Travis 网页中默认界面只能看到最后跑的一个任务，而未打 Tag 的任务又会报

>Skipping a deployment with the releases provider because this is not a tagged commit 

这曾让我一度以为自己的脚本哪里写错了，但是又找不到错误原因……

### 自动发布 APK 到 fir.im

自动发布到 Github 对于开发人员已经足够，但是考虑到项目实际需要以及国情，还是有必要选择一个国内的 App 分发服务，fir.im、蒲公英都是不错的选择，不但允许游客下载，还提供了二维码等更适合对接手机的功能，国内下载速度也很快。由于 fir.im 提供了比较方便的 CLI 工具，因此本文以 fir.im 为例，在`.travis.yml`中添加以下几行：

```yaml
before_install:
- gem install fir-cli
after_deploy:
- fir p app/build/outputs/apk/app-release.apk -T $FIR_TOKEN -c "`git cat-file tag $TRAVIS_TAG`"
```

即在环境构建阶段安装 fir-cli，在发布成功后通过 fir 命令行工具将 apk 上传到 fir。

其中`$FIR_TOKEN`可以在 fir.im 的用户->API Token 中找到，然后在 Travis CI 控制台中创建环境变量`FIR_TOKEN`并粘贴即可。

这里有个小技巧，如果我们仅仅上传 APK 文件到 fir.im，看到链接的测试人员其实并不知道这次发布所包含的变动，因此通过`git cat-file tag $TRAVIS_TAG`将当前发布 tag 所包含的附加信息一同上传了。其中`$TRAVIS_TAG`变量是 Travis CI 每次运行自动附带的环境变量，还有很多其他的[Travis 环境变量](https://docs.travis-ci.com/user/environment-variables/)供我们玩出更多花样。


### 发布完毕后自动发邮件通知

虽然 Travis CI 也有通知功能，但不能定制模板，通知内容也仅仅为提示 CI 运行的结果，显然更适合开发人员。我们还是希望最终能以更友好的方式通知团队成员，同时考虑到邮件送达率，可以优先选择如[Submail](http://submail.cn/)、[SendCloud](http://sendcloud.sohu.com/)等国内邮件发送服务。

这里以 Submail 为例，首先需要在 Submail 内创建邮件模板，比如我们可以创建这样一封触发式邮件模板：

```plain
Hi 亲

@var(TRAVIS_REPO_SLUG)新版本@var(TRAVIS_TAG)已经发布了，功能更新：

@var(TAG_DESCRIPTION)

去下载：
http://fir.im/w13s
```

创建后可以得到邮件模板 id，根据 Submail 手册，将模板中所需要的变量置入，最终可以使用一行 Curl 指令发送一封邮件：

```yaml
after_deploy:
  - curl -d "appid=10948&to=allo.vince@gmail.com&subject=[自动通知] 安卓新版本$TRAVIS_TAG发布&project=u2c0r2&signature=$SUBMAIL_SIGN&vars={\"TRAVIS_REPO_SLUG\":\"$TRAVIS_REPO_SLUG\",\"TRAVIS_TAG\":\"$TRAVIS_TAG\",\"TAG_DESCRIPTION\":\"$(git cat-file tag $TRAVIS_TAG | awk 1 ORS='<br>')\"}" https://api.submail.cn/mail/xsend.json
```

其中 Submail 用到的认证凭据 signature 同样是通过 Travis CI 控制台配置的。

### 总结

最终完成的示例项目[在此](https://github.com/AlloVince/evandroid)。其实所有的 yml 文件配置不到 30 行，就能省去繁琐的日常工作，何乐而不为呢。最后回顾一下自动化后的日常工作：

提交代码：

```bash
git add .
git commit -m "这里是注释"
git push origin
```

打 Tag

```bash
git tag -a v0.0.1-alpha.1 -m "这里是Tag注释，说清楚这个版本的主要改动，也可以省略-m参数直接写长文本"
git push origin --tags
```

如果发现打错了 tag，可以删除本地及远程 tag

```bash
git tag -d v0.0.1-alpha.1
git push origin --delete tag v0.0.1-alpha.1
```

大部分 Tag 标签虽然仅用于内测，但是仍然建议遵循[版本语义化](http://semver.org/lang/zh-CN/)原则。

#### References

- [Travis CI User Documentation](https://docs.travis-ci.com/)
- [用 Travis CI 给 Android 项目部署 Github Release](http://kescoode.com/travis-ci-android-github-release/)

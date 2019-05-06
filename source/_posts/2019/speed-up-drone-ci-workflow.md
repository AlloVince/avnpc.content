---  
title: 容器环境持续集成优化，Drone CI 提速 500%  
s: speed-up-drone-ci-workflow  
date: 2019-05-06 14:16:24  
published: true  
tags:   
 - drone  
 - ci  
 - semantic release  
 - docker
---

前文介绍了[容器环境下 Drone + semantic release 实现的语义化持续集成 Workflow](https://avnpc.com/pages/drone-gitflow-kubernetes-for-cloud-native-ci)，为了方便演示，流程仅给出了工作流中最重要的几个环节，实际用起来可能会发现不少值得优化的地方。

因此本次在这个工作流的基础上，介绍一些[容器环境下 CI 的优化及提速方法](https://avnpc.com/pages/speed-up-drone-ci-workflow)，方法本身不限定一定要使用 Drone，使用同样的思路完全可以套用到其他的 CI 工具中。

## 优化前项目概况

以一个生产环境的实际项目为例，项目的主要结构如下

```plain
├── Dockerfile
├── dist/
├── node_modules/
├── package.json
└── src/
```

这是一个比较常见的基于 React 的前端项目，用 `npm list | wc -l` 可以看到有 3952 个依赖，项目会通过 webpack 打包到 `dist`目录下，打包命令被封装成 `npm run build`。最终 `dist` 目录通过 `Dockerfile` 被打包到 Nginx 的 Docker 镜像内， 生产环境直接运行打包后的镜像即可。

Dockerfile 是这样编写的

```dockerfile
FROM node:10 as build  
WORKDIR /app  
COPY . /app  
RUN npm install  
RUN npm run build  

FROM nginx:1.15-alpine  
COPY --from=build /app/dist /usr/share/nginx/html  
```

使用了 [Docker 的多阶段构建](https://docs.docker.com/develop/develop-images/multistage-build/)功能，即 npm 的安装，编译作为第 1 个阶段，编译完成后仅将编译的结果 `dist` 文件夹复制出来，其余未复制的文件丢弃，这样打包后的镜像仅为 23.2MB，更利于部署。

## 流程瓶颈分析

发布流程直接套用前文介绍的 [Gitflow + semantic release 工作流](https://github.com/AlloVince/drone-ci-demo)。可以看到此时的一次发布是比较慢的，push 到 master 构建 staging 镜像用时 9:18，semantic release 打上 Tag 构建 production 镜像用时 6:24。

![](https://static.avnpc.com/blog/2019/drone-speedup-before.png)

这个过程中到底慢在哪里呢， 在 Drone 的构建过程中看到，容器构建的耗时占了 90%以上，一方面 npm 需要下载安装 3000 多个依赖，另一方面 webpack 的编译也需要 30s 左右，如果网络再有不稳定，等待时间无疑会更长。

![](https://static.avnpc.com/blog/2019/drone-speedup-before-detail.png)

另一个耗时的元凶也很明显，由于引入了 semantic release， push master 和 release 两个动作会触发 2 次 CI，每次 CI 都进行了 Docker 镜像的构建，但其实如果没有异常发生，两个 Docker 镜像对应的其实是同一份代码，应当是完全一致的，即 release 时的镜像构建所花费的时间是浪费的。

其他当然还有应用层面的优化，比如可以用 yarn 替代 npm，使用更快的源，去除不必要的依赖等等，但这些并不在本文的讨论范围内，就略过不提。

## 引入缓存减少重复下载

每次构建都要下载 3000 多个依赖，那么最容易想到的当然是将这些依赖缓存起来，但是在这个项目中，npm 下载/编译都发生在容器构建环节，这是比较难引入缓存的。因此首先要做的，是将下载/编译过程从容器转移到 CI，通过 CI 完成下载/编译，再将结果复制到容器镜像内。

在下载/编译转移到 CI 的基础上，可以直接使用 Drone 提供的缓存插件，目前根据不同文件系统，Drone 可选的缓存插件有

- [Volume Cache](http://plugins.drone.io/drillster/drone-volume-cache/)
- [AWS S3 Cache](http://plugins.drone.io/drone-plugins/drone-s3-cache/)
- [SFTP Cache](http://plugins.drone.io/appleboy/drone-sftp-cache/)
- [Google Cloud Storage Cache](http://plugins.drone.io/hvalle/drone-gcs-cache/)

这里以 Volume Cache 为例，`.drone.yml`如下。这里的语法对应 Drone-v1.0 以上版本，可能与官方部分旧文档有出入。

``` yaml
steps:
- name: restore-cache  
  image: drillster/drone-volume-cache  
  settings:  
    restore: true  
    mount:  
      - ./.npm-cache  
      - ./node_modules  
  volumes:  
    - name: cache  
      path: /cache   

- name: npm-install  
  image: node:10  
  commands: 
    - npm config set cache ./.npm-cache --global  
    - npm install  
 
- name: build-dist  
  image: node:10  
  commands:  
    - npm run build

- name: rebuild-cache  
  image: drillster/drone-volume-cache  
  settings:  
    rebuild: true  
    mount:  
      - ./.npm-cache  
      - ./node_modules  
  volumes:  
    - name: cache  
      path: /cache

volumes:  
  - name: cache  
    host:  
      path: /tmp/cache
```

Volume Cache 插件使用很简单，首先需要声明一个 Volume，对应主机的一个文件夹，这里使用的是`/tmp/cache`。Volume Cache 插件的参数中，`mount` 列出需要缓存的文件夹，`restore: true`会将文件从主机复制到容器，因此放在 pipeline 的开头，`rebuild: true`则反之，放在 pipeline 最后。

另外注意使用 Volume 需要在 Drone 中将 Repo 设置为 Trusted。

而此时的 Dockerfile 就只剩下文件复制的部分了

```dockerfile
FROM nginx:1.15-alpine  
COPY ./dist /usr/share/nginx/html  
```

在增加了缓存后，构建的时长大幅下降到 2:38，整体耗时下降了 50%以上。

![](https://static.avnpc.com/blog/2019/drone-speedup-after-detail1.png)

## 通过 Docker Tag 省略重复的构建

在上文的基础上，不难想到 push master 和 release 造成的重复构建，是否也可以同样通过缓存去除。这当然在理论上也是可行的，但是由于缓存并不稳定，因此需要更为通用的方法。

在 semantic release 的流程中，push master 和 release 的唯一区别就是 release 增加了一个 git tag。而 git tag 本质上只是对一个特定 commit 的引用，并不会改变 commit 记录，因此 push master 和 release 两次触发的 CI 中，最后一次 commit 是相同的，即 `DRONE_COMMIT_SHA` 不会改变。

基于这一点，我们可以在 push master 的构建中，将`DRONE_COMMIT_SHA`作为 Docker 镜像额外的 Tag，在  release 环节，只要给有 `DRONE_COMMIT_SHA` Tag 的镜像再打上最终的版本号 Tag 即可，并不需要在 release 环节从头构建镜像。

这个过程对应 `.drone.yml` 如下

``` yaml
  - name: push-docker-staging  
    image: plugins/docker  
    settings:  
      repo: allovince/xxx
      username: allovince  
      password: 
        from_secret: DOCKER_PASSWORD  
      tag:  
        - staging  
        - sha_${DRONE_COMMIT_SHA} 
    when:  
      branch: master  
      event: push  
  
  - name: semantic-release  
    image: gtramontina/semantic-release:15.13.3  
    environment:  
      GITHUB_TOKEN:  
        from_secret: GITHUB_TOKEN  
    entrypoint:  
      - semantic-release  
    when:  
      branch: master  
      event: push  
  
  - name: push-docker-production  
    image: plugins/docker  
    environment:  
      DOCKER_PASSWORD:  
        from_secret: DOCKER_PASSWORD  
    commands:   
      - docker -v  
      - nohup dockerd &  
      - docker login -u allovince -p $${DOCKER_PASSWORD}  
      - docker pull allovince/xxx:sha_$${DRONE_COMMIT_SHA}  
      - docker tag allovince/xxx:sha_$${DRONE_COMMIT_SHA} allovince/xxx:$${DRONE_TAG}  
      - docker push allovince/xxx:$${DRONE_TAG}  
    when:  
      event: tag  
    privileged: true
```

假设最后一次 commit 的 hash 是 `c0558777`， release 版本是 v1.0.9， push master 后， 镜像将打上

- staging
- sha_c0558777

两个 Tag，在 release 后，镜像将再增加一个 `v1.0.9`的 Tag。

需要注意的是为镜像打 Tag 使用到了 Docker-in-Docker，需要 privileged flag，即`privileged: true`

同时 docker tag 等命令依赖 docker daemon 的启动，否则会报错

> Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?

一种方式是挂载主机的 daemon `/var/run/docker.sock`，另一种方式是在容器内启动 docker daemon，我这里使用的是后者，对应 `nohup dockerd &`，而在 release 阶段，由于任务仅仅是为 docker 镜像额外增加一个 tag，以及通知生产环境发布，因此上文中的 cache 等环节都可以通过条件省略，结果如下。

![](https://static.avnpc.com/blog/2019/drone-speedup-after-detail2.png)

如此优化后 release 环节的耗时缩短到 1 分钟以内，看看最终成果，从代码提交到发布完成，总耗时不到 5 分钟，是比较友好的。

![](https://static.avnpc.com/blog/2019/drone-speedup-after.png)

---  
title: 使用 WebStorm Debug 基于 TypeScript 或 Babel 的 node.js 项目  
s: webstorm-debug-typescript-babel-project  
date: 2019-04-02 15:41:37  
published: true  
tags:  
 - WebStorm  
 - Typescript  
 - Babel 
 - Debug
---  
  
WebStorm 有很方便的 Debug 功能，如果是普通 node.js 项目的话，参考 [WebStorm 官方的 Debug 说明](https://www.jetbrains.com/help/webstorm/running-and-debugging-node-js.html) 配置即可。  
  
由于 node.js 7+ 后都使用 [Inspector](https://nodejs.org/en/docs/guides/debugging-getting-started/) 来实现 Debug， 因此需要注意最低保证 node.js 版本 >= 7 及 JetBrains WebStorm 版本 >= 2017.1+。  
  
鉴于 js 项目的多样性，实际项目中，经常会使用 Babel 或者 TypeScript 编译，特别记录下 [WebStorm 在使用 TypeScript 或 Babel 时的 Debug 配置](https://avnpc.com/pages/webstorm-debug-typescript-babel-project)以备忘。 下文以 node.js v10 及 WebStorm 2019.1 为例。  
  
  
## WebStorm Debug 基于 Babel 的项目  
  
虽然 babel 有 [babel-node](https://babeljs.io/docs/en/babel-node) 可以直接运行未编译代码，但并不推荐在 Debug 中使用 babel-node 直接替换 node。这是由于 babel-node [只是一个 node cli 的简单封装](https://github.com/babel/babel/blob/master/packages/babel-node/src/babel-node.js)，在 babel-node 的一些早期版本中， 并未加入 `--inspect-brk` 等 Debug 所需参数的支持，可能会引发无法打断点或 Debug 进程无法退出等问题。

首先确认好 babel 的安装情况

对于 Babel 7

```bash
npm install --save-dev @babel/core @babel/cli @babel/register
```

对于 Babel 6

```bash
npm install --save-dev babel-core babel-cli babel-register
```

推荐的 Debug 配置如下：

1. 打开 `Run/Debug Configuration` 窗口
2. 新建一个配置，类型选择 `Node.js`
3. `Node Interpreter` 选择本地安装的 node 路径
4. `Node Parameters` 中通过 `-r` 参数，在 node 启动时额外加载 babel 的运行时， 即
   -  如果是 Babel 7, 填入 `-r @babel/register`
   -  如果是 Babel 6， 填入 `-r babel-register`
5. `Working Directory` 中填入当前项目的根目录
6. `JavaScript file` 中填入要 Debug 的 js 文件的相对路径 （相对项目根目录）。如果是类似 Express 之类的 Web 服务，填入服务启动入口文件相对路径

如下图

![WebStorm debug Babel](https://static.avnpc.com/blog/2019/webstorm_debug_babel.png)

## WebStorm Debug 基于 TypeScript 的项目  

和 Babel 类似， TypeScript 的项目也可以用同样的思路进行 Debug

```bash
npm install --save-dev typescript ts-node
```

配置如下

1. 打开 `Run/Debug Configuration` 窗口
2. 新建一个配置，类型选择 `Node.js`
3. `Node Interpreter` 选择本地安装的 node 路径
4. `Node Parameters` 填入 `-r ts-node/register`
5. `Working Directory` 中填入当前项目的根目录
6. `JavaScript file` 中填入要 Debug 的 ts 文件的相对路径

如图

![WebStorm debug TypeScript](https://static.avnpc.com/blog/2019/webstorm_debug_typescript.png)

---
title: 简单分析webpack的热更新
date: 2022-03-21
tags:
 - webpack
categories:
 - 技术文章
---

## 序言

webpack的热更新我一直都在用，但是很少深入去了解它的具体原理。最近看到极客时间的webpack课程，正好有机会深入解析一下webpack的热更新。

## 热更新和文件监听

基于WDS (Webpack-dev-server)的模块热替换，只需要局部刷新页面上发生变化的模块，同时可以保留当前的页面状态，比如复选框的选中状态、输入框的输入等。当代码有修改时自动构建，构建完成后通过热更新让浏览器去变化，省去了自己刷新浏览器的步骤。

热更新需要借助`webpack-dev-server` 和 `HotModuleReplacementPlugin`来完成

输出文件在内存中，没有磁盘IO，所以构建速度更快一点

与之形成对比的是文件监听，也就是 `webpack --watch`，它输出的文件放在磁盘中。这里顺便说一下监听的原理：

> 轮询去判断文件的最后编辑时间是否发生了变化，当发生变化时，先缓存起来，等待一段时间（aggregateTimeout），在这个时间段内的所有变化文件拿去构建，一起把构建的结果生成到bundle中

## 基本用法

1. 在package.json 中 添加命令

```json
"scripts": {
    "dev": "webpack-dev-server --open" // open 参数每次构建完成自动开启浏览器
}

```

2. 创建webpack development config

3. 配置插件，这个插件是webpack内置的，所以不需要引用

```js
plugins: [
  new webpack.HotModuleReplacementPlugin()
]

```

4. 配置热更新

```js
devServer: {
  contentBase: './dist', // 服务的基础目录
  hot: true // 开启热更新
}

```
## 原理分析

首先来看看webpack的几个基本概念：

- Webpack Compile： 将 JS 源代码编译成 bundle.js

- HMR Server: 用来将热更新的文件输出给 HMR Runtime

- Bundle Server： 提供文件在浏览器的访问，以服务的方式访问

- HMR Runtime：会被注入到浏览器，更新文件的变化

- bundle.js 构建好输出的文件


webpack 在热更新模式下，启动服务后，服务端会与客户端建立一个长链接。（用websocket）

文件修改后，就会触发编译, 生成新的hash值。

所以说存在一个监听本地代码的变化方法，主要是通过setupDevMiddleware方法实现的。
这个方法主要执行了webpack-dev-middleware库，调用了compiler的watch。

服务端会通过长链接向客户端推送一条消息，客户端收到后，会重新请求一个 js 文件。

返回的 js 文件会调用 webpackHotUpdatehmr 方法，用于替换掉 __webpack_modules__ 中的部分代码。


## webpack-dev-middleware和webpack-dev-server的区别

其实就是因为webpack-dev-server只负责启动服务和前置准备工作，所有文件相关的操作都抽离到webpack-dev-middleware库了，主要是本地文件的编译和输出以及监听，无非就是职责的划分更清晰了。


[轻松理解webpack热更新原理](https://juejin.cn/post/6844904008432222215#heading-1)

[Webpack 热更新原理](https://segmentfault.com/a/1190000040382502)

[轻松理解webpack热更新原理](https://juejin.cn/post/6844904008432222215)


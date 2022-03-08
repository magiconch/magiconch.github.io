---
title: 如何用webpack打包umd模块并测试打包结果
date: 2019-02-22
tags:
 - webpack
categories:
 -  技术文章
---

对于 JavaScript 的模块而言, webpack 可以用来build 基于浏览器或服务端的包.


下面让我们学习如何使用webpack生成UMD.

首先需要全局安装webpack

`npm install -g webpack`

让我们先来创建一个用来返回两数之和的加法模块.
```js
// add.js
module.exports = function add(a, b) {
  return a + b;
};

接下来,我们来建立webpack配置

// webpack.config.js
module.exports = {
  entry: './add.js',
  output: {
    filename: './dist/add.js',
    // export to AMD, CommonJS, or window
    libraryTarget: 'umd',
    // the name exported to window
    library: 'add'
  }
};

```
接下来使用webpack来build你的模块.
```cmd
$ webpack
Hash: 87657f97293582af3ac3
Version: webpack 4.29.3
Time: 435ms
Built at: 02/22/2019 9:01:57 AM
   Asset      Size  Chunks             Chunk Names
./add.js  1.17 KiB       0  [emitted]  main
Entrypoint main = ./add.js
[0] ./add.js 83 bytes {0} [built
```
现在你可以来使用CommonJS, AMD, script tag这三种不同的方式来测试你的UMD包是否正确了.

## CommonJS

在测试之前,需要确定是否安装成功Nodejs环境,

当你使用webpack打包的程序中包含了调用Window的内容时(比如操作dom啥的),需要将commonJS转换成浏览器可识别的代码.这一步有很多方法,就我而言,我推荐你使用browserify,
它的logo贼好看,让我有种在写咒语的感觉.

而且也很好用,你只要在terminal下输入 `browserify 想要转换的文件 > 生成文件`,就可以生成可以在浏览器环境下使用的js啦.

如果你不想详细测试,也不想装browserify,还有一种偷懒的办法可以不完整的测试你的代码, 在nodejs环境下定义 `global.window = {};`,代码应该也可以运行.

$ node 
```js
> var add = require('./dist/add');
> add(1, 2);
```
## AMD

AMD模块需要一个requirejs模块,我这里采用的是在cdn上引用,你也可以把它下载下来,自己引入试一下.需要注意的是,如果自己引用的话,需要注意文件路径.

下载链接在[这里](https://requirejs.org/docs/download.html)

[AMD (中文版)](https://github.com/amdjs/amdjs-api/wiki/AMD-(%E4%B8%AD%E6%96%87%E7%89%88)
```html
<!-- amd.html -->
<html>
<body>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/require.js/2.3.2/require.min.js"></script>
  <script>
    window.requirejs(['dist/add'], function(add) {
      console.log(add(1, 2));
    });
  </script>
</body>
</html>
```
## Script Tag

这个就是最简单的全局暴露,没啥好说的.
```html
<!-- script-tag.html -->
<html>
<body>
  <script src="./dist/add.js"></script>
  <script>
    console.log(window.add(1, 2));
  </script>
</body>
</html>
```

本文参考了[Build to UMD with webpack@1](https://remarkablemark.org/blog/2016/09/22/webpack-build-umd/)

感谢他救我于水火之中
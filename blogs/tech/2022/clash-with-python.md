---
title: clash for windows 使用pip出现ProxyError问题分析
date: 2022-04-09
tags:
 - python
categories:
 -  技术文章
---

我查到的方法是开启specify protocol或者设置成System Proxy Type = PAC

我现在使用的是v0.18.10版本，亲测可用.

问题解决了，那么为什么设置成PAC就可以？换句话说，为什么pip会报这个错误？

我查到了一个这样的内容[this](https://note.bobo.moe/2021/02/clash-for-windows-pip-proxyerror.html)

这里还有一个坑就是如果你要使用最近更新的v0.19以后的版本，可能还需要改一下配置信息，具体可以看一下[官方的issue](https://github.com/Fndroid/clash_for_windows_pkg/issues/2588)


---
title: 分析笔试平台存在的问题
date: 2022-03-22
tags:
 - 面试
categories:
 -  技术文章
---

今天惠风和畅，天气晴朗，我收到了一个笔试邀请。

是一个叫 `showmebug`的网站制作的，他的编辑功能一言难尽，但是你却没有办法离开屏幕。

每次离开屏幕都会显示一个alert, 你说你在线环境做的不好吧，还不让人切屏出去debug，有没有天理了。

![alert](/in.png)


勇敢的小旱獭当然不能害怕，我就在想怎么样把他的离开页面给关掉

本来还在想是不是需要用到一些js的事件来阻止，结果打开控制台一看。

好家伙，这就是直接把切屏数量发送到了后台,我本来以为他会在最后再统一上传，结果是每次切屏都会上传。

![alert](/in2.png)

然后来看看我们要拦截的这个url。

![alert](/in3.png)

我们可以看到，中间这段应该就是考试号，只需要把发送到这个url的请求全给他拦截了就行。

现在随便一个chrome拦截http请求的插件，就结束了。

## 技术总结

下面开始技术总结，当我们写一个在线笔试系统的时候，需要做到尽可能好用，其次关于有可能作弊的一系列信息，建议加密后传输。

最后，我在想，他调用摄像头应该不会是实时监控吧，这样带宽太大了，应该是截屏照片的模式监控，那么我能不能每次上传一张吴彦祖的照片上去？




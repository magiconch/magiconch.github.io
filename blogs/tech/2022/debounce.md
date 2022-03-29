---
title: 从lodash源码中学习防抖函数
date: 2022-03-26
tags:
 - 源码阅读
 - 前端
categories:
 - 技术文章
---

函数防抖是一个很常用的优化手段。我往常写防抖函数都是这样写的：

```js
function debounce(func, wait) {
    var timeout;
    return function () {
        clearTimeout(timeout) 
        timeout = setTimeout(func, wait);
    }
}
```


但是我今天在debug的时候，偶然进入到了Lodash模块的防抖函数，发现他的代码很长。他到底做了什么呢？


```js
function debounce(func, wait, options) {
  var lastArgs,
      lastThis,
      maxWait,
      result,
      timerId,
      lastCallTime,
      lastInvokeTime = 0,
      leading = false,
      maxing = false,
      trailing = true;

  if (typeof func != 'function') {
    throw new TypeError(FUNC_ERROR_TEXT);
  }
  wait = toNumber(wait) || 0;
  if (isObject(options)) {
    leading = !!options.leading;
    maxing = 'maxWait' in options;
    maxWait = maxing ? nativeMax(toNumber(options.maxWait) || 0, wait) : maxWait;
    trailing = 'trailing' in options ? !!options.trailing : trailing;
  }

  function cancel() {
      // ...
  }

  function flush() {
      // ...
  }

  debounced.cancel = cancel;
  debounced.flush = flush;
  return debounced;
}
```

先来看看大概的代码内容

他提供了很多Option，大概分析一下这些option的作用

| Argument | Meaning |
|-----|------|
| lastArgs | 上次函数的参数 |
| lastThis | 上次调用的This |
| lastCallTime |  |
| lastInvokeTime | |
| timerId | 计时器ID |
| maxWait | 允许func最长延迟时间 |
| result | |
| leading | 指定超时的前驱调用 |
| trailing | 指定在超时后的后续调用 |

首先执行的是cancel函数，没啥好说的，就是清空执行栈

```js
  function cancel() {
    if (timerId !== undefined) {
      clearTimeout(timerId);
    }
    lastInvokeTime = 0;
    lastArgs = lastCallTime = lastThis = timerId = undefined;
  }
```
然后是flush方法

```js
function flush() {
    return timerId === undefined ? result : trailingEdge(now());
  }
```

这里如果没有生成计时器ID，就用Option里面自带的result, 或者创建一个新的的计时器

> 但是这块我不理解的是，Option里面已经有了timerId，为啥还要再引入一个result.

```js
  function trailingEdge(time) {
    timerId = undefined;

    // 只有当 lastArgs 存在的时候才会被调用
    // 因为这时候func已经至少调用了一次防抖函数
    if (trailing && lastArgs) {
      return invokeFunc(time);
    }
    lastArgs = lastThis = undefined;
    return result;
  }
```

这个invokeFunc函数的作用就是执行上一次的函数调用，然后把函数调用的结果返回。


```js
  function invokeFunc(time) {
    var args = lastArgs,
        thisArg = lastThis; 

    lastArgs = lastThis = undefined;
    lastInvokeTime = time;
    result = func.apply(thisArg, args);
    return result;
  }
```

接下来就是最后一句代码`return debounced` 中的 `debounced`函数

也是最重要的环节：

```js
function debounced() {
    var time = now(),
        isInvoking = shouldInvoke(time);

    lastArgs = arguments;
    lastThis = this;
    lastCallTime = time;

    if (isInvoking) {
      if (timerId === undefined) {
        return leadingEdge(lastCallTime);
      }
      if (maxing) {
        // Handle invocations in a tight loop.
        clearTimeout(timerId);
        timerId = setTimeout(timerExpired, wait);
        return invokeFunc(lastCallTime);
      }
    }
    if (timerId === undefined) {
      timerId = setTimeout(timerExpired, wait);
    }
    return result;
  }

```

这块的逻辑还是很好理解的。
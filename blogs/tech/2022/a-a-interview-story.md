---
title: 小旱獭的面试之路（二）
date: 2022-03-16
tags:
 - 面试
categories:
 -  技术文章
---

## 引言

今天惠风和畅，天气晴朗，掐指一算，果然又是面试的日子。

果不其然，在某招聘软件上，我就收到了面试官发来的面试题。

一共题目有三道，先来看看第一道：


## 请使用js基础代码写一个通用方法实现深克隆，不使用既有快捷方法

```js
function DeepClone(obj) {
    if (obj === null || typeof (obj) !== 'object' || 'isActiveClone' in obj)
        return obj;

    if (obj instanceof Date)
        var temp = new Date(obj);
    else
        var temp = obj.constructor();
    for (let key in obj) {
        if (Object.hasOwnProperty.call(obj, key)) {
            obj['isActiveClone'] = null;
            temp[key] = DeepClone(obj[key]);
            delete obj['isActiveClone'];
        }
    }
    return temp;
}

```

这道题其实还有很多衍生的东西可以谈，比如说深拷贝成功之后如何测试深拷贝的结果，也就是DeepEqual。当然了，谈到深拷贝，我们也需要判断以下他是否是浅拷贝，由此可以再衍生一个方法，shallowEqual。


## 求两个数组的交集

不多说，直接set开整

```ts
const arr1 = [1,2,3,4,5];
const arr2 = [4,5,6,7,8];

function intersection(arr1: number[], arr2: number[]) {
  const a1: Set<number> = new Set(arr1);

  const a2: Set<number> = new Set(arr2);

  return Array.from(
    new Set([...a1]
      .filter(x => a2.has(x)))
    );
}

```

## 要实现一个功能：多级组织机构的展示，后端给的接口数据+示例：

```js
[
    {id: 1, name: '”开发部”', parentId: 0},
    {id: 2, name: '部门1', parentId: 1},
    {id: 3, name: '部门2', parentId: 1},
    {id: 4, name: '部门3', parentId: 3},
    {id: 5, name: '部门4', parentId: 4},
]

```

这道题我只能说这不巧了吗，我前几天才看过类似的题。

这个应该分为两个部分

- 将多级组织转化为树形视图
- 绘制树形视图

这其实都是一个递归的思路。当然了，他们应该也是可以转化为遍历的。

本次面试的代码我放在了这个仓库里[this](https://github.com/magiconch/demo/tree/main/interview)


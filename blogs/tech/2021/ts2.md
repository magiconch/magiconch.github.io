---
title: 在TypeScript中，keyof的若干用法
date: 2021-12-19
tags:
 - typescript
categories:
 -  技术文章
---

## 引言

keyof

keyof 与 Object.keys 略有相似

```ts
interface Point {
    x: number;
    y: number;
}

// type keys = "x" | "y"
type keys = keyof Point;

```
这里的keyof取interface的所有键。那么keyof有什么常见的用法吗？

我们可以来看一下这个例子：

```ts
const cat = {
    breed: "Persian",
    age: 3
}

function getName(cat: object, name: string) {
    return cat[name];
}
```
这是不使用keyof和泛型写出来的get方法。它存在几个很难避免的缺陷

* 返回值无法确认，这里如果name是未预期的值，ts无法直接进行类型校验
* 无法约束key

这时候就可以考虑用keyof来加强函数的类型功能。如下所示：

```ts
function getProperty<T extends object, K extends keyof T>(o: T, name: K): T[K] {
  return o[name]
}

const person: Person = {
  age: 22,
  name: "Tobias",
};

// name is a property of person
// --> no error
const name = getProperty(person, "name");

// gender is not a property of person
// --> error
const gender = getProperty(person, "gender");

```
这段代码通过泛型很巧妙的将get方法抽离出来。通过枚举T来达到类型检查。非常非常巧妙。

## keyof any 是什么意思？

我们在阅读一些源码中，可以看到 keyof any 这样的用法。顾名思义就是将any枚举出来。

> 对没错，我说的就是Record

```ts
/**
 * Construct a type with a set of properties K of type T
 */
type Record<K extends keyof any, T> = {
    [P in K]: T;
};
```
这里的 keyof any 是指的所有类型吗？实际上它仅仅包括了`string | number | symbol`，

我们可以这样测试一下

```ts
let a: any;
a['a'] //ok
a[0] // ok
a[Symbol()] //ok
a[{}] // error
```
由此我们可以得出Record的合法使用途径：

```ts
type t0 = Record<1, string> // { 1: string }
type t1 = Record<"prop", string> // { prop: string }
type t3 = Record<string, string> // { [name: string]: string }
type t4 = Record<number, string> // { [name: number]: string }
type t5 = Record<{a : string}, string> // error
```

> 友情提示：在设置了 keyOfStringOnly=true 的情况下 number也会报错








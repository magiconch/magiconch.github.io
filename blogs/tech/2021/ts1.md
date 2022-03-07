---
title: Typescript中的Record到底是什么？
date: 2021-12-17
tags:
 - typescript
categories:
 -  技术文章
---

## 引言

这是ts2.1 引入的新概念

>Record<Keys， Type>
>Constructs an object type whose property keys are Keys and whose >property values are Type. This utility can be used to map the >properties of a type to another type.” — TypeScript’s documentation
> 
> 译：构造一个对象类型，Keys 表示对象的属性键 、Type 表示对象的属性值，用于将一种类型属性映射到另一种类型

我们来看一下官方给出的用法：

```ts
interface CatInfo {
  age: number;
  breed: string;
}
 
type CatName = "miffy" | "boris" | "mordred";
 
const cats: Record<CatName, CatInfo> = { // Record
  miffy: { age: 10, breed: "Persian" }, 
  boris: { age: 5, breed: "Maine Coon" },
  mordred: { age: 16, breed: "British Shorthair" },
};
cats.boris;
```
那么这里和索引签名的区别在哪里？

```ts
interface CatInfo {
  age: number;
  breed: string;
}
type studentScore = { [ name: string ]: CatInfo }

```

似乎Record的用法更加简明，语义也更加的清晰。除此之外他还有什么用法呢？

## 统一修改属性类型

```ts
interface Staff {
  name:string,
  salary:number,
}
  
 type StaffJson = Record<keyof Staff, string>

  const product: StaffJson = {
    name: 'John',
    salary:'3000'
  }

```
这段代码我们将Staff的所有属性都转化为了String，由此生成了一个新的类型。

## 与Partial协同

```ts
type seniorRole = 'manager' | 'leader'
type technicalRole = 'developer' | 'engineer'
const benefits: Partial<Record<seniorRole, 'Free Parking'> & Record<technicalRole, 'Free Coffee'>> = {};


benefits.manager = 'Free Parking';
benefits.developer = 'Free Parking';//ERROR: no free parking for dev
```
> Parrtial的作用是将所有的类型都修改为可选。



这段代码可能理解起来有些困难，我们可以拆看看一下：

`Record<seniorRole, 'Free Parking'> & Record<technicalRole, 'Free Coffee'>`

这里面的&代表交叉类型，也就是说我们的管理层可以免费停车，普通员工可以免费恰咖啡。这两种权益同时存在。

那么加上Partial呢，就代表<管理层, 免费停车>和<普通员工, 恰咖啡> 都是可选的。

通过 Record、Partial 和 Intersection 类型一起工作，它创建了一个强类型的benefits对象，并在键和值类型之间建立关联。

强类型对象使得在编译时更容易捕获错误，使 IDE 在键入时更容易标记错误，并可以提供代码补全。

---
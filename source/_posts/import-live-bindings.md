---
title: 具名导入是解构么
date: 2023-03-06 10:37:02
categories: JavaScript
tags: ES6
excerpt: 在这篇文章中，讲解了具名导入和解构赋值的区别。
author: pengfei.zuo
---

## 解构赋值

```JavaScript
let obj = {
    n: 1,
    increase() {
        this.n ++;
    }
}

let {n, increase} = obj;

console.log(n)// 1
increase();
console.log(n)// 1
```

![](https://raw.githubusercontent.com/retech-fe/image-hosting/main/img/2023/03/06/10-11-48-c9ffa534710aeda4c39533dbc991dd44-20230306101148-da3953.png)

+ 上面的底阿妈在做解构赋值
+ 解构赋值的英文全称是： Destructuring assignment
+ 解构赋值 => 解构 + 赋值；重点是赋值
+ 对于obj中的n是基本数据类型，解构出来obj中n的值是1，然后赋值给一个新的变量n，他的值是拷贝过来的，在内存中是独立的两份内存空间
+ 对于obj中的increase是引用类型，increase的值是存放函数的堆内存中的地址，然后拷贝赋值给一个新的变量increase，也是两块栈内存空间，但是指向的同一份堆内存中的函数
+ increase();改变的是obj的n，对于结构出来的n没有影响

## 具名导入

+ 具名导入不是解构赋值

myMoudule.mjs

```JavaScript
export let n = 1;
export function increase() {
    n++;
}
```
test.mjs

```JavaScript
import {n, increase} from './myMoudule.mjs'; // 不是解构赋值

console.log(n); // 1
increase();
console.log(n); // 2
```

## live bindings

![](https://raw.githubusercontent.com/retech-fe/image-hosting/main/img/2023/03/06/10-28-18-f525f3c5fa11fe8a6a123c0402edf755-20230306102817-6f0270.png)

+ 具名导入是live bindings
+ 具名导入是给导出的内容取的新的名字n和increase，他们用的内存空间还是模块内部的内存空间
+ 当我们调用increase去改变n的时候，还是改变的是模块内部的n；具名导入的值也会受到影响


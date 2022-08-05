---
title: JavaScript中new操作符的原理
date: 2022-08-05 19:08:02
categories: JavaScript基础
tags: JavaScript
excerpt: 主要讲解JavaScript中new操作符的原理
author: taotao1.liu
---


## new的用处

new的作用是通过构造函数来创建一个实例对象，该实例与原型和构造函数之间的关系如下图所示：

![](https://raw.githubusercontent.com/retech-fe/image-hosting/main/img/2022/08/05/18-48-49-e655bf8be2097ba1f24e62b5a22c2922-202171393058303-04e34a.jpeg)

## 具体步骤

javascript中的new是一个语法糖，new的过程如下

1. 在内存中创建一个新对象
2. 这个新对象内部的[[Prototype]]指针被赋值为构造函数的 prototype 属性
3. 构造函数内部的 this 被赋值为这个新对象（即 this 指向新对象）

> 很多直接把下面4.5的归为一步

4. 执行构造函数内部的代码（给新对象添加属性）
5. 如果构造函数返回非空对象，则返回该对象；否则，返回刚创建的新对象




## 模拟 new 操作符

### 方案一

```
function _new(constructor, ...arg) {
    // ① 创建一个新的空对象 obj
    const obj = {};
    // ② 将新对象的的原型指向当前函数的原型
    obj.__proto__ = constructor.prototype;
    // ③ 新创建的对象绑定到当前this上
    const result = constructor.apply(obj, arg);
    // ④ 如果没有返回其他对象，就返回 obj，否则返回其他对象
    return typeof result === 'object' ? result : obj; // 为什么要判断返回值的类型？这是因为new一个实例的时候，如果没有return，就会根据构造函数内部this绑定的值生成对象，如果有返回值，就会根据返回值生成对象，为了模拟这一效果，就需要判断apply后是否有返回值。
}
function Foo(name) {
    this.name = name;
}
var luckyStar = _new(Foo, 'luckyStar');
console,log(luckyStar.name); //luckyStar

```

### 方案二

```
function _new(fn, ...arg) {
    // 基于函数原型创建一个新的对象
    const obj = Object.create(fn.prototype);
    const newObj = fn.apply(obj, arg);
    return newObj instanceof Object
}
    
function GirlName(name, age) {
    this.name = name;
    this.age = age;
    this.sayName = function () {
        console.log(this.name);
    };
}
const xiaoMei = _new(GirlName, 'Xiao Mei', 18)
console.log(xiaoMei) 
```

### 方案三

```
function New() {
    let obj = {}; // 创建对象
    console.log(arguments);
    let constructor =  [].shift.call(arguments); // 获取构造函数
    console.log(arguments);
    if (constructor.prototype !== null) {
        obj.__proto__ = constructor.prototype; // 构造函数链接到新对象
    }
    // let ret = constructor.apply(obj, [].slice.call(arguments)); // 改变this指向 但是对参数使用slice会阻止某些JavaScript引擎中的优化
    let ret = constructor.apply(obj, (arguments.length === 1 ? [arguments[0]] : Array.apply(null, arguments))); // 替代方案
    console.log(arguments);
    console.log(typeof ret);
    if ((typeof ret === "object" || typeof ret === "function") && ret !== null) {
        return ret;
    }
    return obj; // 如果函数没有返回对象类型Object(包含Functoin, Array, Date, RegExg, Error)，那么new表达式中的函数调用将返回该对象引用。
}

function name(a, b) {
 this.a = a;
 this.b = b;
}

let c = New(name, 1, 2)
let d = new name(1, 2)
console.log(c);
console.log(d);
```

## 为什么要判断返回值的类型？

这是因为new一个实例的时候，如果没有return，就会根据构造函数内部this绑定的值生成对象，如果有返回值，就会根据返回值生成对象，为了模拟这一效果，就需要判断apply后是否有返回值。

```javascript
function GirlName(name, age) {
    this.name = name;
    this.age = age;
    this.sayName = function () {
        console.log(this.name);
    };

    // 如果这里 return 'Tomboy'，最后的结果还是 'Xiao Mei'
    return {
        name: 'Tomboy',
        age: 30,
        sayName: function() {
            console.log(this.name)
        }
    }
}

```

```javascript
let xiaoMei = new GirlName('Xiao Mei', 18);
xiaoMei.sayName(); // Tomboy，这里的对象就成了构造函数返回的对象
```

---
title: 继承
date: 2022-07-21 22:11:02
categories: 基础技术之继承
tags: javascript
excerpt: 主要讲解继承中原型和原型链继承和原型、原型链的基本知识
---

# 1. 什么是继承：

- JS 中一切皆对象，必须有一种机制，把所有的对象联系起来，实现类似的“继承”机制。
- 不同于大部分面向对象语言，ES6 之前并没有引入类（class）的概念，JS 并非通过类而是通过构造函数来创建实例，javascript 中的继承是通过原型链来体现的。
- 其基本思想是利用**原型让一个引用类型继承另一个引用继承的属性和方法**。

## **2. 为什么要继承**

> 提高代码的重用性、较少代码的冗余

刚刚提到："利用原型让一个引用类型继承另一个引用继承的属性和方法",那么咱们先看下 原型，原型链相关的知识

**2.1 原型（prototype）**

> JS 中所有函数都会有 prototype 属性，只有函数才有其所有的属性和方法都能被构造函数的实例对象共享访问

代码如下：

```javascript
	function Person(name){
				this.name = name
	}
    Person.prototype.sayHello(){
        console.log('sayHello')
    }
    let p1 = new Person();
    let p2 = new Person();
    console.log(p1.sayHello) //sayHello
    console.log(p2.sayHello) //sayHello
```

**2.2 构造函数（constructor）**

> JS 中 constructor 存在每个函数的 prototype 属性中，其保存了指向该函数的引用

```javascript
Person.prototype.constructor == Person //true
```

**2.3 原型链**

> 在 JavaScript 中是通过 prototype 对象指向父类对象，直到指向 Object 对象为止（person → Person → Object），这样就形成了一个原型指向的链条，专业术语称之为原型链

- 当我们访问对象的一个属性或方法时，它会先在对象自身中寻找，如果有则直接使用，如果没有则会去原型对象中寻找，如果找到则直接使用。
- 如果没有则去原型的原型中寻找,直到找到 Object 对象的原型，Object 对象的原型没有原型，如果在 Object 原型中依然没有找到，则返回 undefined。
- 注意：Object 的*proto*为空, 即原型链的尽头一般来说都是 Object.prototype

```javascript
p1.__proto__ == Person.prototype
```

> JS 引擎查找摸个属性时，先查找对象本身是否存在该属性，如果不存在就会在原型链上一层一层进行查找

![在这里插入图片描述](https://img-blog.csdnimg.cn/f182141e627e498ea0aafea81cc5ff4d.png)

**由图我们可知几个关系：**

- 每一个构造函数都有(原型)prototype 指向它的原型对象。
- 原型对象有 constructor 指向它的构造函数。
- 构造函数可以通过 new 的创建方式创建实例对象
- 实例对象通过 proto 指向它的原型对象。
- 原型对象也有自己的原型对象，通过 proto 指向。

## **3. 目前我总结常用的一共有 6 种继承方式**

- 原型链继承
- 借用构造函数继承
- 组合式继承（原型链+构造函数）
- 原型式继承
- 寄生式继承
- 寄生组合式继承
  咱们本期先讲解 **原型链继承**和**原型式继承** ，如果想了解其他的继承可查看我的 关于 [详解 js 继承的那些事儿](https://blog.csdn.net/qq_34574204/article/details/120716964)

```javascript
//父类
function Person(name) {
  this.name = name
  this.sum = function () {
    alert("this.name", this.name)
  }
}
Person.prototype.age = 100
```

**3.1 原型链继承**

**实现方式：** 让实例的原型等于父类的实例

**优点：** 实例可以继承父类的构造个函数，实例的构造函数，父类的原型

**缺点：** 不能向父类传递参数，由于实例的原型等于父类的实例，那么改变父类的属性，实例的属性也会跟着改变

```javascript
function child() {
  this.name = "xiaoming"
}
child.prototype = new Person()
let child1 = new Child()
child1.name //xiaoming
child1.age //100
child1 instanceof Person //true
```

**3.2 原型式继承**

**实现方式：** 函数包装对象，返回对象的引用，这个函数就变成可以随时添加实例或者对象，Object.create()就是这个原理

**优点：** 复用一个对象用函数包装

**缺点：** 所有实例都继承在原型上面 无法复用

```javascript
function child(obj) {
  function F() {}
  F.prototype = obj
  return new F()
}
let child1 = new Person()
let child2 = child(child1)
child2.age //100
```

---
title: 手撕JavaScript深拷贝&浅拷贝
date: 2023-03-01 18:13:02
categories: JavaScript
tags: 源码
excerpt: 在这篇文章中，处理了深拷贝中一般考虑不到的正则、Set、Map、Function、Date、Error等数据类型。
author: pengfei.zuo
---

## 深拷贝要考虑到所有数据类型

```JavaScript
var obj = {
    name: 'name',
    age: 30,
    bool: true,
    n: null,
    u: undefined,
    symb: Symbol('symbol'),
    big: BigInt(12),
    arr: [1, 2, 3],
    reg: /^\d+$/,
    fn: function() {
        console.log('fn')
    },
    time: new Date(),
    err: new Error('error'),
    set: new Set([{a: 1}, {a: 2}]),
    map: new Map([[{a: 1}, {b: 1}], [{a: 2}, {b: 2}]]),
    child: {
        ele: 'body',
        x: 100
    }
};

// 测试循环依赖
// obj.obj = obj;
```
+ 7基本数据类型：string、number、boolean、null、undefined、symbol、BigInt
+ symbol类型做value、symbol类型做key
+ 引用类型：数组、正则、日期、Error、Set、Map
+ 函数类型
+ 深层次对象类型
+ 本文没有考虑`ArrayBuffer`这种引用数据类型；lodash的深拷贝中会处理

## 使用展开运算符完成对象拷贝存在的问题

![](https://raw.githubusercontent.com/retech-fe/image-hosting/main/img/2023/03/01/17-47-45-1c64aafdd8991d06b87f184a7706827c-20230301174744-3ee2f1.png)

+ 展开运算符可以把obj的每一项展开然后赋值给obj2

![](https://raw.githubusercontent.com/retech-fe/image-hosting/main/img/2023/03/01/14-05-21-83736599276fecc03505b6b6db4841a5-20230301140521-775aec.png)

+ 因为obj2在堆内存中开辟了一份新的内存空间来存储从obj拷贝过来的所有字段的值
+ obj2 === obj => false，因为是一个新的对象，所以不相等


![](https://raw.githubusercontent.com/retech-fe/image-hosting/main/img/2023/03/01/17-49-01-e9f74cd0ed48c8ac4caab069d5ccd585-20230301174901-e7c13d.png)

+ obj2中的child和arr又是引用类型，内部还包含其他基本数据类型或者引用类型
+ obj2.child === obj.child => false
+ **...展开运算符是浅拷贝**，只拷贝了obj的第一级的key对应的值给到obj2，第二级以及后面的第三级、第四级没有进行拷贝；所以obj.child 和 obj2.child指向的是同一块内存

## 使用JSON.stringfy进行深拷贝存在的问题

![](https://raw.githubusercontent.com/retech-fe/image-hosting/main/img/2023/03/01/14-22-12-a193581dea878c06fee76d93ef98e990-20230301142211-7fc8c8.png)

+ BigInt类型的值不能被序列化

![](https://raw.githubusercontent.com/retech-fe/image-hosting/main/img/2023/03/01/17-43-50-05d483678394b67055eb34525ba338cd-20230301174349-c2651f.png)

+ 把BigInt类型的值先去掉，就能使用JSON.stringfy来深拷贝
+ undefined类型的值丢了
+ 函数类型的值丢了
+ Symbol类型的值丢了
+ 正则类型的值变成空对象了
+ Error类型的值变成空对象了
+ Set类型的值变成空对象了
+ Map类型的值变成空对象了
+ key是Symbol类型的话，该key都丢了，即使自己for in循序也不能遍历Symbol类型的key
+ Date日期类型的值变成字符串了

> JSON.stringify会把值变成字符串，对于特殊的数据类型就是转不了字符串在反序列化回来，所以就会存在问题

![](https://raw.githubusercontent.com/retech-fe/image-hosting/main/img/2023/03/01/14-45-23-b1a19c3ae0136ba365968428a5b600ef-20230301144522-6abb1c.png)

+ JSON.parse(JSON.stringify(obj))先stringify再parse得到的对象确实是深拷贝
+ 第二级child也进行深拷贝了

## 第一版深拷贝实现

```JavaScript
 function deepClone(obj){
    if(obj == null) return obj;
    if(typeof obj !=='object') return obj;
    let cloneObj = new obj.constructor
    for(let key in obj){
        if(obj.hasOwnProperty(key)){
            cloneObj[key] = deepClone(obj[key])
        }
    }
    return cloneObj
}
```

### 基本思路

+ 如果是基本数据类型直接把原始的值赋给新的克隆后的对象相应的key
+ 如果是引用类型的值，就new一个新的该引用类型对象，然后值赋给新的克隆后的对象相应的key

### 存在的问题

![](https://raw.githubusercontent.com/retech-fe/image-hosting/main/img/2023/03/01/17-56-38-5b983431fd83dfe37a5a831f8ce10e5e-20230301175638-071a44.png)

+ 正则表达式的内容丢失
+ error对象的message丢失
+ set对象长度为0
+ map对象长度为0
+ function函数类型的值因为typeof返回的是'function'并不是'object'，所以直接返回了

## 第二版深拷贝实现

```JavaScript
function deepClone(target){
           
    // 数据类型
    let type = Object.prototype.toString.call(target).slice(8, -1).toLowerCase();

    // 处理null和undefined, null == undefined
    if (target == null) return target;

    // constructor构造函数， 必须要放
    let ctr = target.constructor;

    // Regexp正则和Date日期类型
    if (/^(regexp|date)$/i.test(type)) {
        return new ctr(target);
    }
    // Error类型
    if (/^(error)$/i.test(type)) {
        return new ctr(target.message);
    }
    // Function类型
    if (/^(function)$/i.test(type)) {
        return function proxy(...args) {
            target.call(this, args);
        };
    }
    // Set类型
    if (/^(set)$/i.test(type)) {
        let cloneSet = new ctr;
        target.forEach((item) => {
            cloneSet.add(_deepClone(item))
        })
        return cloneSet;
    }

    // Map类型
    if (/^(map)$/i.test(type)) {
        let cloneMap = new ctr;
        target.forEach((item, key) => {
            cloneMap.set(_deepClone(key), _deepClone(item))
        })
        return cloneMap;
    }

    // Symbol不用处理，因为Symbol(xxx)调用后返回的值不相等
    // 其他数据类型原样返回，后续有处理的其他数据类型如ArrayBuffer就可以在上面继续添加
    if (!/^(array|object)$/i.test(type)) return target;

    let cloneTarget = new ctr
    for(let key in target){
        if(target.hasOwnProperty(key)){
            cloneTarget[key] = deepClone(target[key]);
        }
    }
    return cloneTarget;
}
```

![](https://raw.githubusercontent.com/retech-fe/image-hosting/main/img/2023/03/01/15-45-53-bd2937f7f0265f01ffc751c11ba6297f-20230301154552-7f1208.png)

+ 正则类型的值只需要把原本的正则target传给新的构造函数就不会丢失，而且还创一个新的实例
+ Error类型需要把message传给构造函数
+ Funtion类型需要Object.prototype.toString.call(target).slice(8, -1).toLowerCase()来拿到类型的相信信息，从而对函数类型的值单独处理，不能直接返回

## 第三版深拷贝实现-循环依赖

```JavaScript
 var obj = {
    name: 'name',
    age: 30,
    bool: true,
    n: null,
    u: undefined,
    symb: Symbol('symbol'),
    big: BigInt(12),
    arr: [1, 2, 3],
    reg: /^\d+$/,
    fn: function() {
        console.log('fn')
    },
    time: new Date(),
    err: new Error('error'),
    set: new Set([{a: 1}, {a: 2}]),
    map: new Map([[{a: 1}, {b: 1}], [{a: 2}, {b: 2}]]),
    child: {
        ele: 'body',
        x: 100
    }
};

// 循环依赖
obj.obj = obj;
```

在控制台可以无限查看obj，这就是循环依赖

![](https://raw.githubusercontent.com/retech-fe/image-hosting/main/img/2023/03/01/16-05-10-16c7f6526f8df81edebae1afcca9264a-20230301160509-754980.png)

如果沿用原来的代码会报调用次数太多，爆栈 Maximum call stack size exceeded

![](https://raw.githubusercontent.com/retech-fe/image-hosting/main/img/2023/03/01/16-29-57-de359543a8b33ecb52b96dd468a1163a-20230301162957-f3b6cd.png)

### 解决思路

+ 避免对象或者数组类型的值循环递归

#### 方案一

+ 利用缓存，如果某个对象或者数组类型的值已经处理过就直接返回，然后就不会进入无限递归了
+ 采用WeakMap，是因为弱引用，使用完就被垃圾回收了，避免递归内存溢出


```JavaScript
function deepClone(target, map = new WeakMap()){
           
    // 数据类型
    let type = Object.prototype.toString.call(target).slice(8, -1).toLowerCase();

    // 处理null和undefined, null == undefined
    if (target == null) return obj;

    // constructor构造函数， 必须要放
    let ctr = target.constructor;

    // Regexp正则和Date日期类型
    if (/^(regexp|date)$/i.test(type)) {
        return new ctr(target);
    }
    // Error类型
    if (/^(error)$/i.test(type)) {
        return new ctr(target.message);
    }
    // Function类型
    if (/^(function)$/i.test(type)) {
        return function proxy(...args) {
            target.call(this, args);
        };
    }
    
    // Set类型
    if (/^(set)$/i.test(type)) {
        let cloneSet = new ctr;
        target.forEach((item) => {
            cloneSet.add(_deepClone(item))
        })
        return cloneSet;
    }

    // Map类型
    if (/^(map)$/i.test(type)) {
        let cloneMap = new ctr;
        target.forEach((item, key) => {
            cloneMap.set(_deepClone(key), _deepClone(item))
        })
        return cloneMap;
    }

    // Symbol不用处理，因为Symbol(xxx)调用后返回的值不相等
    // 其他数据类型原样返回，后续有处理的其他数据类型如ArrayBuffer就可以在上面继续添加
    if (!/^(array|object)$/i.test(type)) return target;

    // map中有就直接返回，不要递归了，就不会无限递归了
    if(map.get(target)) return map.get(target);

    let cloneTarget = new ctr;
    // 存放对象或数组类型值的拷贝结果
    map.set(target, cloneTarget);
    for(let key in target){
        if(target.hasOwnProperty(key)){
            cloneTarget[key] = deepClone(target[key], map);
        }
    }
    return cloneTarget;
}

```

#### 方案二

+ 利用WeakSet或数组来存放某个对象或者数组类型的target，发现处理过就直接返回
+ 直接返回

```JavaScript
function deepClone(target, set = new WeakSet()){
           
    // 数据类型
    let type = Object.prototype.toString.call(target).slice(8, -1).toLowerCase();

    // 处理null和undefined, null == undefined
    if (target == null) return obj;

    // constructor构造函数， 必须要放
    let ctr = target.constructor;

    // Regexp正则和Date日期类型
    if (/^(regexp|date)$/i.test(type)) {
        return new ctr(target);
    }
    // Error类型
    if (/^(error)$/i.test(type)) {
        return new ctr(target.message);
    }
    // Function类型
    if (/^(function)$/i.test(type)) {
        return function proxy(...args) {
            target.call(this, args);
        };
    }
    
    // Set类型
    if (/^(set)$/i.test(type)) {
        let cloneSet = new ctr;
        target.forEach((item) => {
            cloneSet.add(_deepClone(item))
        })
        return cloneSet;
    }

    // Map类型
    if (/^(map)$/i.test(type)) {
        let cloneMap = new ctr;
        target.forEach((item, key) => {
            cloneMap.set(_deepClone(key), _deepClone(item))
        })
        return cloneMap;
    }

    // Symbol不用处理，因为Symbol(xxx)调用后返回的值不相等
    // 其他数据类型原样返回，后续有处理的其他数据类型如ArrayBuffer就可以在上面继续添加
    if (!/^(array|object)$/i.test(type)) return target;

    // map中有就直接返回，不要递归了，就不会无限递归了
    // 返回target
    // if(set.has(target)) return target;
    // 直接return
    if(set.has(target)) return;

    let cloneTarget = new ctr;
    // 存放对象或数组类型值的拷贝结果
    set.add(target);
    for(let key in target){
        if(target.hasOwnProperty(key)){
            cloneTarget[key] = deepClone(target[key], set);
        }
    }
    return cloneTarget
}
```
+ 直接返回就相当于没有拷贝

![](https://raw.githubusercontent.com/retech-fe/image-hosting/main/img/2023/03/01/16-51-51-edc0dafd8350f96358585b840723aa34-20230301165150-8eb9d3.png)

+ 返回target，相当于没拷贝

![](https://raw.githubusercontent.com/retech-fe/image-hosting/main/img/2023/03/01/16-54-01-0f8ffae364eac2fafb7cf081ff952919-20230301165401-4911cb.png)


### 优化传递map

```JavaScript
function deepClone(obj){
    let map = new WeakMap();
    const _deepClone = (target) => {
        // 数据类型
        let type = Object.prototype.toString.call(target).slice(8, -1).toLowerCase();

        // 处理null和undefined, null == undefined
        if (target == null) return obj;

        // constructor构造函数， 必须要放
        let ctr = target.constructor;

        // Regexp正则和Date日期类型
        if (/^(regexp|date)$/i.test(type)) {
            return new ctr(target);
        }
        // Error类型
        if (/^(error)$/i.test(type)) {
            return new ctr(target.message);
        }
        // Function类型
        if (/^(function)$/i.test(type)) {
            return function proxy(...args) {
                target.call(this, args);
            };
        }
        
          // Set类型
	    if (/^(set)$/i.test(type)) {
	        let cloneSet = new ctr;
	        target.forEach((item) => {
	            cloneSet.add(_deepClone(item))
	        })
	        return cloneSet;
	    }
	
	    // Map类型
	    if (/^(map)$/i.test(type)) {
	        let cloneMap = new ctr;
	        target.forEach((item, key) => {
	            cloneMap.set(_deepClone(key), _deepClone(item))
	        })
	        return cloneMap;
	    }

        // Symbol不用处理，因为Symbol(xxx)调用后返回的值不相等
        // 其他数据类型原样返回，后续有处理的其他数据类型如ArrayBuffer就可以在上面继续添加
        if (!/^(array|object)$/i.test(type)) return target;

        // map中有就直接返回，不要递归了，就不会无限递归了
        if(map.has(target)) return target;
        // if(set.has(target)) return;

        let cloneTarget = new ctr;
        // 存放对象或数组类型值的拷贝结果
        map.set(target, cloneObj);
        for(let key in target){
            if(target.hasOwnProperty(key)){
                cloneTarget[key] = _deepClone(target[key]);
            }
        }
        return cloneTarget;
    }
    return _deepClone(obj);
    
}
```


## 同时支持浅拷贝和深拷贝

```JavaScript
function clone(...params) {
    let target = params[0];
    let deep = false;
    let len = params.length;
    // 存放已经处理过的target的result结果
    let map = params[2] || new WeakMap();
    if (typeof target === 'boolean' && len >= 2) {
        deep = target;
        target = params[1];
    }
    // 发现循环依赖就直接返回，避免进入递归，导致爆栈
    if (map.has(target)) return map.get(target);
    

    // 数据类型
    let type = Object.prototype.toString.call(target).slice(8, -1).toLowerCase();

    // 处理null和undefined, null == undefined
    if (target == null) return target;

    // constructor构造函数， 必须要放
    let ctr = target.constructor;

    // Regexp正则和Date日期类型
    if (/^(regexp|date)$/i.test(type)) {
        return new ctr(target);
    }
    // Error类型
    if (/^(error)$/i.test(type)) {
        return new ctr(target.message);
    }
    // Function类型
    if (/^(function)$/i.test(type)) {
        return function proxy(...args) {
            target.call(this, args);
        };
    }
    
    // Set类型
    if (/^(set)$/i.test(type)) {
        let cloneSet = new ctr;
        target.forEach((item) => {
            cloneSet.add(_deepClone(item))
        })
        return cloneSet;
    }
	
    // Map类型
    if (/^(map)$/i.test(type)) {
        let cloneMap = new ctr;
        target.forEach((item, key) => {
            cloneMap.set(_deepClone(key), _deepClone(item))
        })
        return cloneMap;
    }

    // Symbol不用处理，因为Symbol(xxx)调用后返回的值不相等
    // 其他数据类型原样返回，后续有处理的其他数据类型如ArrayBuffer就可以在上面继续添加
    if (!/^(array|object)$/i.test(type)) return target;

    // 处理对象和数组
    let cloneObj = new ctr;
    // 存放该target对应的结果result
    map.set(target, cloneObj);
    // for in可以同时遍历数组和对象
    for (let key in target) {
        if (target.hasOwnProperty(key)) {
            cloneObj[key] = deep ? clone(true, target[key], map) : target[key];
        }
    }
    return cloneObj;
}

let deepCopy = clone(true, obj);
let copy = clone(obj);
```

## 函数类型的深拷贝

+ 函数拷贝有两种方案
+ 方案一返回一个代理函数，函数内部还是调用原函数，就是本文用的方案
+ 方案二new Function，方案如下

```JavaScript
const getType = (obj) => Object.prototype.toString.call(obj)

const isObject = (target) =>
  (typeof target === "object" || typeof target === "function") &&
  target !== null

const canTraverse = {
  "[object Map]": true,
  "[object Set]": true,
  "[object Array]": true,
  "[object Object]": true,
  "[object Arguments]": true,
}
const mapTag = "[object Map]"
const setTag = "[object Set]"
const boolTag = "[object Boolean]"
const numberTag = "[object Number]"
const stringTag = "[object String]"
const symbolTag = "[object Symbol]"
const dateTag = "[object Date]"
const errorTag = "[object Error]"
const regexpTag = "[object RegExp]"
const funcTag = "[object Function]"

const handleRegExp = (target) => {
  const { source, flags } = target
  return new target.constructor(source, flags)
}

const handleFunc = (func) => {
  // 箭头函数直接返回自身
  if (!func.prototype) return func
  const bodyReg = /(?<={)(.|\n)+(?=})/m
  const paramReg = /(?<=\().+(?=\)\s+{)/
  const funcString = func.toString()
  // 分别匹配 函数参数 和 函数体
  const param = paramReg.exec(funcString)
  const body = bodyReg.exec(funcString)
  if (!body) return null
  if (param) {
    const paramArr = param[0].split(",")
    return new Function(...paramArr, body[0])
  } else {
    return new Function(body[0])
  }
}

const handleNotTraverse = (target, tag) => {
  const Ctor = target.constructor
  switch (tag) {
    case boolTag:
      return new Object(Boolean.prototype.valueOf.call(target))
    case numberTag:
      return new Object(Number.prototype.valueOf.call(target))
    case stringTag:
      return new Object(String.prototype.valueOf.call(target))
    case symbolTag:
      return new Object(Symbol.prototype.valueOf.call(target))
    case errorTag:
    case dateTag:
      return new Ctor(target)
    case regexpTag:
      return handleRegExp(target)
    case funcTag:
      return handleFunc(target)
    default:
      return new Ctor(target)
  }
}

const deepClone = (target, map = new WeakMap()) => {
  if (!isObject(target)) return target
  let type = getType(target)
  let cloneTarget
  if (!canTraverse[type]) {
    // 处理不能遍历的对象
    return handleNotTraverse(target, type)
  } else {
    // 这波操作相当关键，可以保证对象的原型不丢失！
    let ctor = target.constructor
    cloneTarget = new ctor()
  }

  if (map.get(target)) return target
  map.set(target, true)

  if (type === mapTag) {
    //处理Map
    target.forEach((item, key) => {
      cloneTarget.set(deepClone(key, map), deepClone(item, map))
    })
  }

  if (type === setTag) {
    //处理Set
    target.forEach((item) => {
      cloneTarget.add(deepClone(item, map))
    })
  }

  // 处理数组和对象
  for (let prop in target) {
    if (target.hasOwnProperty(prop)) {
      cloneTarget[prop] = deepClone(target[prop], map)
    }
  }
  return cloneTarget
}
```




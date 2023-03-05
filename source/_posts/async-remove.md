---
title: 消除异步的传染性
date: 2023-03-05 21:37:02
categories: JavaScript
tags: ES6
excerpt: 在这篇文章中，讲解了什么是异步传染性及如何解决该问题。
author: pengfei.zuo
---

## 什么是异步传染性？

```JavaScript
//  封装了一个ajax请求
//  内部使用了await，所以函数也要标记为async
async function doRequest() {
    //  fetech请求是异步，所以要用await
    await fetch('https://jsonplaceholder.typicode.com/todos/1').then((resp) => resp.json())
}

// m1调用了doRequest函数，doRequest是async函数，所以也得标记async
async function m1() {
    return await doRequest();
}

// m2调用了m1 函数，m1是async函数，所以也得标记async
async function m2() {
    return await m1();
}

// m3调用了m2函数，m2是async函数，所以也得标记async
async function m3() {
    return await m2();
}

```

+ fetch这个节点是异步的造成了连锁反应
+ 一个节点发生了异步，这个节点的所有调用链路全部都得异步
+ 函数式编程的场景，m1、m2、m3就不是纯函数了，全部变成了副作用
+ 

## 目标

+ 去掉所有的async await，改成同步调用
+ 直接返回结果（ps: 异步的怎么会马上、立刻、直接返回结果呢？）

```JavaScript
function doRequest() {
    return fetch('https://jsonplaceholder.typicode.com/todos/1');
}

function m1() {
    return doRequest();
}

function m2() {
    return m1();
}

function m3() {
    return m2();
}

function main() {
	const res = m3();
	console.log(res);
}
```

## 怎么消除异步？

+ 异步肯定不能马上、立刻、直接返回结果
+ 只有报错这一条路
+ 把promise作为异常抛出

![](https://raw.githubusercontent.com/retech-fe/image-hosting/main/img/2023/03/05/19-06-08-6ca06f1f65321258ed4389befc8b2e30-20230305190607-4bc15a.png)

+ 函数开始执行，调用fetch发送请求
+ 不用等待fetch请求的返回结果，直接把fetch请求返回的promoise实例作为异常抛出
+ fetch请求是异步的，需要把真正请求的返回值缓存起来，供后面的逻辑使用
+ 对真正的要执行的函数用try-catch包裹，捕获刚刚抛出的异常
+ 如果捕获的异常对象是Promise的实例， 就监听其成功或失败状态，成功或者失败后重新发起请求
+ 重新发起请求，会从缓存中直接获取上次请求缓存下的结果

```JavaScript
function doRequest() {
    return fetch('https://jsonplaceholder.typicode.com/todos/1');
}

function m1() {
    return doRequest();
}

function m2() {
    return m1();
}

function m3() {
    return m2();
}

function main() {
    const res = m3();
    console.log(res);
}

function run(func) {
    // 缓存
    let cache = [];
    // 请求索引
    let i = 0;
    // 原生fetch
    let _originFetch = window.fetch;
    // 劫持fetch函数-代理原生fetch
    // 新fetch的核心功能： 1. 发送请求 2.报错
    window.fetch = (...args) => {
        if (cache[i]) {
            if (cache[i].status === 'fullfilled') {
                return cache[i].data;
            } else if (cache[i].status === 'rejected') {
                throw cache[i].err;
            }
        }

        const result = {
            status: 'pending',
            data: null,
            err: null,
        }
        cache[i++] = result;

        // 发送异步请求
        const promise = _originFetch(...args)
            .then(resp => resp.json())
            .then(resp => {
                result.status = 'fullfilled';
                result.data = resp;
            }, err => {
                result.status = 'rejected';
                result.err = err;
            });
        
        
        // 马上报错， 让函数强制执行完毕，不用等异步回来才往下执行
        // 把原生fetch的promise作为错误直接抛出去，让fetch马上结束
        throw promise;
    }

    // 劫持原生fetch函数报错后，就会try-catch住
    try {
        // 运行传入的函数
        func();
    } catch (err) {
        if (err instanceof Promise) {
            const reRun = () => {
            	    // index 置零
                i = 0;
                // 重新发起请求
                func()
            }
            	// 异步请求成功或者失败都重新执行原来的函数
            err.then(reRun, reRun);
        }
    }
    
}

run(main);

```

## React中的应用

```JavaScript

const userResource = get	UserResource();

function ProfilePage () {
	return  (
		<Suspense fallback={<h1>Loading...</h1>}>
			<ProfileDetails />
		</Suspense>
	)
}

function ProfileDetails () {
	console.log(1)
	// 直接throw一个Promise则一直处于loading状态
	// throw new Promise();
	const user = userResource.read();
	return <h1>{user.name}</h1>
}

```

+ Suspense里面子组件ProfileDetails的状态，请求中就显示loading，返回结果后就显示用户详情
+ React中ProfileDetails组件没有返回Promise，就是一个普通的组件
+ 1会打印两次
+ 和上面的原理一样，先执行ProfileDetails，会打印第一次，直接throw一个promoise的异常
+ 成功后继续再执行一次，然后打印第二次






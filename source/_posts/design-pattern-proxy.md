---
title: 设计模式之代理模式
date: 2022-11-04 13:34:02
categories: design pattern
tags: design pattern
excerpt: 在这篇文章中，对设计模式中的代理模式从定义、类图、应用场景、及和适配器模式和装饰器模式的对比进行了详细的阐释。
author: pengfei.zuo
---

## 动机

+ 由于一个对象不想或者不能直接引用另外一个对象，所以需要通过通过一个称之为“代理”的第三者来实现间接引用
+ 代理模式就是为目标对象创造一个代理对象，在客户端和目标对象之间起到中介的作用
+ 这样就可以在代理对象里增加一些逻辑判断、调用前或调用后执行一些操作，从而实现了扩展目标的功能
+ 并且可以通过代理对象去掉客户不能看到的内容和服务或者添加客户需要的额外服务

通过引入一个新的对象（如小图片和远程代理对象）来实现对真实对象的操作或者将新的对 象作为真实对象的一个替身，这种实现机制即为代理模式，通过引入代理对象来间接访问一 个对象，这就是代理模式的模式动机。

## 定义

代理模式(Proxy Pattern) ：给某一个对象提供一个代理，并由代理对象控制对原对象的引用。代理模式的英 文叫做Proxy或Surrogate，它是一种对象结构型模式。

### 生活中的案例： 

火车票代购、房产中介、律师、海外代购、明星经纪人

## 类图和时序图

代理模式包含如下角色：

+ Subject: 抽象主题角色
+ Proxy: 代理主题角色
+ RealSubject: 真实主题角色

### 类图

![](https://raw.githubusercontent.com/retech-fe/image-hosting/main/img/2022/11/02/10-26-01-64c8ea716b6d0418d34098b70e28a01f-20221102102601-0c2235.png)

### 时序图

![](https://raw.githubusercontent.com/retech-fe/image-hosting/main/img/2022/11/02/10-26-49-30dd6b97fd498e2e30966ecda612ce7d-20221102102649-34451f.png)


## 一个例子-明星经纪人

![](https://raw.githubusercontent.com/retech-fe/image-hosting/main/img/2022/11/02/10-31-06-8884ec170ae11170f68b856bd3a7cd74-20221102103105-9cb7fa.png)

```
abstract class Star {
    abstract answerPhone(): void;
}

class Angelababy extends Star {
    public available: boolean = true;
    answerPhone(): void {
        console.log('你好,我是Angelababy.');
    }
}
class AngelababyAgent extends Star {
    constructor(private angelababy: Angelababy) {
        super();
    }
    answerPhone(): void {
        console.log('你好,我是Angelababy的经纪人.');
        if (this.angelababy.available) {
            this.angelababy.answerPhone();
        }
    }
}
let angelababyAgent = new AngelababyAgent(new Angelababy());
angelababyAgent.answerPhone();
```

## 场景

### 事件委托代理

+ 事件捕获指的是从document到触发事件的那个节点，即自上而下的去触发事件
+ 事件冒泡是自下而上的去触发事件
+ 绑定事件方法的第三个参数，就是控制事件触发顺序是否为事件捕获。true为事件捕获；false为事件冒泡,默认false。

![](https://raw.githubusercontent.com/retech-fe/image-hosting/main/img/2022/11/02/10-34-07-84f93fe717c67717f1e3140a88843e60-20221102103407-3b1502.png)


```HTML
<body>
    <ul id="list">
        <li>1</li>
        <li>2</li>
        <li>3</li>
    </ul>
<script>
  let list = document.querySelector('#list');
  list.addEventListener('click',event=>{
       alert(event.target.innerHTML);
  });     
</script>    
</body>

```

### 虚拟代理(图片预加载)

#### app.js

```JavaScript
let express=require('express');
let path=require('path')
let app=express();
app.get('/images/loading.gif',function (req,res) {
    res.sendFile(path.join(__dirname,req.path));
});
app.get('/images/:name',function (req,res) {
    setTimeout(() => {
        res.sendFile(path.join(__dirname,req.path));
    }, 2000);
});
app.get('/',function (req,res) {
    res.sendFile(path.resolve('index.html'));
});
app.listen(8080);
```
####  index.html 

```HTML
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
    <style>
        .bg-container {
            width: 600px;
            height: 400px;
            margin: 100px auto;
        }

        .bg-container #bg-image {
            width: 100%;
            height: 100%;
        }
    </style>
</head>

<body>
    <div id="background">
        <button data-src="/images/bg1.jpg">背景1</button>
        <button data-src="/images/bg2.jpg">背景2</button>
    </div>
    <div class="bg-container">
        <img id="bg-image" src="/images/bg1.jpg" />
    </div>
    <script>
        let container = document.querySelector('#background');

        class BackgroundImage {
            constructor() {
                this.bgImage = document.querySelector('#bg-image');
            }
            setSrc(src) {
                this.bgImage.src = src;
            }
        }
        class LoadingBackgroundImage { 
             static LOADING_URL= `/images/loading.gif`;
            constructor() {
                this.backgroundImage = new BackgroundImage();
            }
            setSrc(src) {
                this.backgroundImage.setSrc(LoadingBackgroundImage.LOADING_URL);
                let img = new Image();
                img.onload = () => {
                    this.backgroundImage.setSrc(src);
                }
                img.src = src;
            }
        }
        let loadingBackgroundImage = new LoadingBackgroundImage();
        container.addEventListener('click', function (event) {
            let src = event.target.dataset.src;
            loadingBackgroundImage.setSrc(src + '?ts=' + Date.now());
        });
    </script>
</body>

</html>
```

### 虚拟代理(图片懒加载)

+ 当前可视区域的高度 window.innerHeight || document.documentElement.clientHeight
+ 元素距离可视区域顶部的高度 getBoundingClientRect().top
+ [getBoundingClientRect](https://developer.mozilla.org/zh-CN/docs/Web/API/Element/getBoundingClientRect)
+ DOMRect 对象包含了一组用于描述边框的只读属性——left、top、right 和 bottom，单位为像素。除了 width 和 height 外的属性都是相对于视口的左上角位置而言的


```JavaScript
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Lazy-Load</title>
    <style>
        .image {
            width: 300px;
            height: 200px;
            background-color: #CCC;
        }

        .image img {
            width: 100%;
            height: 100%;
        }
    </style>
</head>

<body>
    <div class="image-container">
        <div class="image">
            <img data-src="/images/bg1.jpg">
        </div>
        <div class="image">
            <img data-src="/images/bg2.jpg">
        </div>
        <div class="image">
            <img data-src="/images/bg1.jpg">
        </div>
        <div class="image">
            <img data-src="/images/bg2.jpg">
        </div>
        <div class="image">
            <img data-src="/images/bg1.jpg">
        </div>
        <div class="image">
            <img data-src="/images/bg2.jpg">
        </div>
        <div class="image">
            <img data-src="/images/bg1.jpg">
        </div>
        <div class="image">
            <img data-src="/images/bg2.jpg">
        </div>
        <div class="image">
            <img data-src="/images/bg1.jpg">
        </div>
        <div class="image">
            <img data-src="/images/bg2.jpg">
        </div>
    </div>
</body>
<script>
    const imgs = document.getElementsByTagName('img');
    const clientHeight = window.innerHeight || document.documentElement.clientHeight;
    let loadedIndex = 0;
    function lazyload() {
        for (let i = loadedIndex; i < imgs.length; i++) {
            if (clientHeight - imgs[i].getBoundingClientRect().top > 0) {
                imgs[i].src = imgs[i].dataset.src;
                loadedIndex = i + 1;
            }
        }
    }
    lazyload();
    window.addEventListener('scroll', lazyload, false);
</script>
</html>
```

### 缓存代理

有些时候可以用空间换时间

#### 正整数的阶乘（factorial）

一个正整数的阶乘（factorial）是所有小于及等于该数的正整数的积，并且0的阶乘为1

```JavaScript
const factorial = function f(num) {
    if (num === 1) {
        return 1;
    } else {
        return (num * f(num - 1));
    }
}

const proxy = function (fn) {
    const cache = {};  // 缓存对象
    return function (num) {
        if (num in cache) {
            return cache[num];   // 使用缓存代理
        }
        return cache[num] = fn.call(this, num);
    }
}

const proxyFactorial = proxy(factorial);
console.log(proxyFactorial(5));
console.log(proxyFactorial(5));
console.log(proxyFactorial(5));
```

#### 斐波那契数列(Fibonacci sequence)

指的是这样一个数列：1、1、2、3、5、8、13、21、34。在数学上，斐波那契数列以如下被以递推的方法定义：F(1)=1,F(2)=1,F(n)=F(n-1)+F(n-2)（n>=3，n∈N*）

```JavaScript
let count = 0;
function fib(n) {
    count++;
    return n <= 2 ? 1 : fib(n - 1) + fib(n - 2);
}
var result = fib(10);
console.log(result, count);//55 110
```

```JavaScript
let count = 0;
const fibWithCache = (function () {
    let cache = {};
    function fib(n) {
        count++;
        if (cache[n]) {
            return cache[n];
        }
        let result = n <= 2 ? 1 : fib(n - 1) + fib(n - 2);
        cache[n] = result;
        return result;
    }
    return fib;
})();
var result = fibWithCache(10);
console.log(result, count);//55 17
```

###  防抖代理

+ 通过防抖代理优化可以把多次请求合并为一次，提高性能
+ 节流与防抖都是为了减少频繁触发事件回调
+ 节流(Throttle)是在某段时间内不管触发了多少次回调都只认第一个，并在第一次结束后执行回调
+ 防抖(Debounce)就是在某段时间不管触发了多少回调都只看最后一个

#### 节流

```HTML
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
    <style>
        #container {
            width: 200px;
            height: 400px;
            border: 1px solid red;
            overflow: auto;
        }

        #container .content {
            height: 4000px;
        }
    </style>
</head>

<body>
    <div id="container">
        <div class="content"></div>
    </div>
    <script>
        function throttle(callback, interval) {
            let last;
            return function () {
                let context = this;
                let args = arguments;
                let now = Date.now();
                if (last) {
                    if (now - last >= interval) {
                        last = now;
                        callback.apply(context, args);
                    }
                } else {
                    callback.apply(context, args);
                    last = now;
                }

            }
        }
        let lastTime = Date.now();
        const throttle_scroll = throttle(() => {
            console.log('触发了滚动事件', (Date.now() - lastTime) / 1000);
        }, 1000);
        document.getElementById('container').addEventListener('scroll', throttle_scroll);
    </script>
</body>

</html>
```
#### 防抖

```HTML
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
    <style>
        #container {
            width: 200px;
            height: 400px;
            border: 1px solid red;
            overflow: auto;
        }

        #container .content {
            height: 4000px;
        }
    </style>
</head>

<body>
    <div id="container">
        <div class="content"></div>
    </div>
    <script>
        function debounce(callback, delay) {
            let timer;
            return function () {
                let context = this;
                let args = arguments;
                if (timer)
                    clearTimeout(timer);
                timer = setTimeout(() => {
                    callback.apply(context, args);
                }, delay);
            }
        }
        let lastTime = Date.now();
        const throttle_scroll = debounce(() => {
            console.log('触发了滚动事件', (Date.now() - lastTime) / 1000);
        }, 1000);
        document.getElementById('container').addEventListener('scroll', throttle_scroll);
    </script>
</body>

</html>
```

#### 防抖案例 -未防抖

```HTML
<body>
    <ul id="todos">
    </ul>
<script>
    let todos = document.querySelector('#todos');
    window.onload = function(){
        fetch('/todos').then(res=>res.json()).then(response=>{
            todos.innerHTML = response.map(item=>`<li "><input value="${item.id}" type="checkbox" ${item.completed?"checked":""}/>${item.text}</li>`).join('');
        });
    }
    function toggle(id){
       fetch(`/toggle/${id}`).then(res=>res.json()).then(response=>{
            console.log('response',response);
        });
    }
    todos.addEventListener('click',function(event){
        let checkbox = event.target;
        let id = checkbox.value;
        toggle(id);
    });
</script>
</body>
```

app.js

```JavaScript
let express=require('express');
let app=express();
app.use(express.static(__dirname));
let todos=[
    {id: 1,text: 'a',completed: false},
    {id: 2,text: 'b',completed: false},
    {id: 3,text: 'c',completed: false},
];
app.get('/todos',function (req,res) {
    res.json(todos);
});
app.get('/toggle/:id',function (req,res) {
    let id=req.params.id;
    todos = todos.map(item => {
        if (item.id==id) {
            item.completed=!item.completed;
        }
        return item;
    });
    res.json({code:0});
});
app.listen(8080);
```

#### 防抖案例 -防抖

todos.html

```HTML
<body>
    <ul id="todos">
    </ul>
    <script>
    let todos = document.querySelector('#todos');
    window.onload = function(){
        fetch('/todos').then(res=>res.json()).then(response=>{
            todos.innerHTML = response.map(item=>`<li "><input value="${item.id}" type="checkbox" ${item.completed?"checked":""}/>${item.text}</li>`).join('');
        });
    }
    function toggle(id){
       fetch(`/toggle/${id}`).then(res=>res.json()).then(response=>{
            console.log('response',response);
        });
    }
    let LazyToggle = (function(id){
        let ids = [];
        let timer;
        return function(id){
            ids.push(id);
            if(timer){
               clearTimeout(timer);
            }
            timer = setTimeout(function(){
                toggle(ids.join(','));
                ids = null;
                clearTimeout(timer);
                timer = null;
            },2000);
        }
    })();
    todos.addEventListener('click',function(event){
        let checkbox = event.target;
        let id = checkbox.value;
        LazyToggle(id);
    });
</script>
```

app.js

```JavaScript
app.get('/toggle/:ids',function (req,res) {
    let ids=req.params.ids;
    ids=ids.split(',').map(item=>parseInt(item));
    todos = todos.map(item => {
        if (ids.includes(item.id)) {
            item.completed=!item.completed;
        }
        return item;
    });
    res.json({code:0});
});
```

### 代理跨域

#### 正向代理
 
+ 正向代理的对象是客户端，服务器端看不到真正的客户端
+ 通过公司代理服务器上网

![](https://raw.githubusercontent.com/retech-fe/image-hosting/main/img/2022/11/02/18-34-19-438745f99bb4afe960201443b2eb9f2c-20221102183418-a1f21d.png)

#### 反向代理

+ 反向代理的对象的服务端,客户端看不到真正的服务端
+ nginx代理应用服务器

![](https://raw.githubusercontent.com/retech-fe/image-hosting/main/img/2022/11/02/18-36-28-b0dad5d0930e95345e79c6d23c6dd414-fanproxy-f1c368.jpeg)


proxy-server.js

```JavaScript
const http = require('http');
const httpProxy = require('http-proxy');
//创建一个代理服务
const proxy = httpProxy.createProxyServer();
//创建http服务器并监听8888端口
let server = http.createServer(function (req, res) {
    //将用户的请求转发到本地9999端口上
    proxy.web(req, res, {
        target: 'http://127.0.0.1:9999'
    });
    //监听代理服务错误
    proxy.on('error', function (err) {
        console.log(err);
    });
});
server.listen(8888, '0.0.0.0');
```

real-server.js

```JavaScript
const http = require('http');
let server = http.createServer(function (req, res) {
    res.end('9999');
});
server.listen(9999, '0.0.0.0');
```

### 代理跨域

+ nginx代理跨域
+ `webpack-dev-server`代理跨域
+ 客户端代理跨域
+ 当前的服务启动在origin(3000端口)上，但是调用的接口在target(4000端口)上
+ [postMessage](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/postMessage)方法可以安全地实现跨源通信
    + otherWindow:其他窗口的一个引用 message:将要发送到其他window的数据
    + message 将要发送到其他window的数据
    + targetOrigin通过窗口的origin属性来指定哪些窗口能接收到消息事件，其值可以是字符串"*"（表示无限制）或者一个URI
  
```JavaScript
otherWindow.postMessage(message, targetOrigin, [transfer]);
````


+ data 从其他window中传递过来的对象
+ origin 调用postMessage时消息发送方窗口的origin
+ source 对发送消息的窗口对象的引用

```JavaScript
window.addEventListener("message", receiveMessage, false);
```
origin.js

```JavaScript
let express=require('express');
let app=express();
app.use(express.static(__dirname));
app.listen(3000);
```

target.js

```JavaScript
let express = require('express');
let app = express();
let bodyParser = require('body-parser');
app.use(bodyParser.urlencoded({ extended: true }));
app.use(express.static(__dirname));
let users = [];
app.post('/register', function (req, res) {
    let body = req.body;
    let target = body.target;
    let callback = body.callback;
    let username = body.username;
    let password = body.password;
    let user = { username, password };
    let id = users.length == 0 ? 1 : users[users.length - 1].id + 1;
    user.id = id;
    users.push(user);
    res.status(302);
    res.header('Location', `${target}?callback=${callback}&args=${id}`);
    res.end();
});
app.listen(4000);
```

reg.html

```HTML
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
</head>

<body>
    <script type="text/javascript">
        window.addEventListener('message', function (event) {
            console.log(event.data);

            if (event.data.receiveId) {
                alert('用户ID=' + event.data.receiveId);
            }
        })
    </script>
    <iframe name="proxyIframe" id="proxyIframe" frameborder="0"></iframe>
    <form action="http://localhost:4000/register" method="POST" target="proxyIframe">
        <input type="hidden" name="callback" value="receiveId">
        <input type="hidden" name="target" value="http://localhost:3000/target.html">
        用户名<input type="text" name="username" />
        密码<input type="text" name="password" />
        <input type="submit" value="提交">
    </form>
</body>

</html>
```

target.html

```HTML
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
</head>

<body>
    <script>
        window.onload = function () {
            var query = location.search.substr(1).split('&');
            let callback, args;
            for (let i = 0, len = query.length; i < len; i++) {
                let item = query[i].split('=');
                if (item[0] == 'callback') {
                    callback = item[1];
                } else if (item[0] == 'args') {
                    args = item[1];
                }
            }
            try {
                window.parent.postMessage({ [callback]: args }, '*');
            } catch (error) {
                console.log(error);
            }
        }
    </script>
</body>

</html>
```

###  $.proxy

+ 接受一个函数，然后返回一个新函数，并且这个新函数始终保持了特定的上下文语境。
+ jQuery.proxy( function, context ) function为执行的函数，content为函数的上下文this值会被设置成这个object对象

```HTML
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>jquery proxy</title>
</head>

<body>
    <button id="btn">点我变红</button>
    <script src="http://libs.baidu.com/jquery/2.0.0/jquery.min.js"></script>
    <script>
        let btn = document.getElementById('btn');
        btn.addEventListener('click', function () {
            setTimeout($.proxy((function () {
                $(this).css('color', 'red');
            }), this), 1000);
        });    
    </script>
</body>

</html>
```

```JavaScript
function proxy(fn, context) {
    return function () {
       return fn.call(context, arguments);
    }
}

```

### Proxy

+ Proxy 用于修改某些操作的默认行为
+ Proxy 可以理解成，在目标对象之前架设一层`拦截`,外界对该对象的访问，都必须先通过这层拦截，因此提供了一种机制，可以对外界的访问进行过滤和改写。
+ Proxy 这个词的原意是代理，用在这里表示由它来`代理`某些操作，可以译为`代理器 `
+  [Proxy](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Proxy)
+ [defineProperty](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty)

```JavaScript
let wang={
    name: 'wanglaoshi',
    age: 29,
    height:165
}
let wangMama=new Proxy(wang,{
    get(target,key) {
        if (key == 'age') {
            return wang.age-1;
        } else if (key == 'height') {
            return wang.height-5;
        }
        return target[key];
    },
    set(target,key,val) {
        if (key == 'boyfriend') {
            let boyfriend=val;
            if (boyfriend.age>40) {
                throw new Error('太老');
            } else if (boyfriend.salary<20000) {
                throw new Error('太穷');
            } else {
                target[key]=val;
                return true;
            }
        }
    }
});
console.log(wangMama.age);
console.log(wangMama.height);
wangMama.boyfriend={
    age: 41,
    salary:3000
}
```

#### Vue2和Vue3

Vue2 中的变化侦测实现对 Object 及 Array 分别进行了不同的处理,Object 使用了 Object.defineProperty API,Array使用了拦截器对 Array 原型上的能够改变数据的方法进行拦截。虽然也实现了数据的变化侦测，但存在很多局限 ，比如对象新增属性无法被侦测，以及通过数组下边修改数组内容，也因此在 Vue2 中经常会使用到 $set 这个方法对数据修改，以保证依赖更新。

Vue3 中使用了 es6 的 Proxy API对数据代理，没有像 Vue2 中对原数据进行修改，只是加了代理包装，因此首先性能上会有所改善。其次解决了 Vue2 中变化侦测的局限性，可以不使用 $set 新增的对象属性及通过下标修改数组都能被侦测到。

## 对比

### 代理模式 VS 适配器模式

适配器提供不同接口，代理模式提供一模一样的接口
 
### 代理模式 VS 装饰器模式 
 
装饰器模式原来的功能不变还可以使用，代理模式改变原来的功能

 
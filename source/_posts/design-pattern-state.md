---
title: 设计模式之状态模式
date: 2022-10-02 10:07:02
categories: design pattern
tags: design pattern
excerpt: 在这篇文章中，对设计模式中的状态模式从定义、类图、代码优化过程、应用场景、优缺点进行了详细的阐释。
author: hongxiang.gao
---


## 定义

状态模式 （State Pattern）允许一个对象在其内部状态改变时改变它的行为，对象看起来似乎修改了它的类，类的行为随着它的状态改变而改变。

+ 对象有自己的状态
+ 不同状态下执行的逻辑不一样
+ 用来减少if...else子句

## 生活中的例子

1）等红绿灯的时候，红绿灯的状态和行人汽车的通行逻辑是有关联的：

+ 红灯亮：行人通行，车辆等待；
+ 绿灯亮：行人等待，车辆通行；
+ 黄灯亮：行人等待，车辆等待；

2）下载文件的时候，就有好几个状态；比如下载验证、下载中、暂停下载、下载完毕、失败，文件在不同状态下表现的行为也不一样，比如

+ 下载中时显示可以暂停下载和下载进度，
+ 下载失败时弹框提示并询问是否重新下载等等。


## 类图

在这些场景中，有以下特点：

+ 对象有有限多个状态，且状态间可以相互切换；
+  各个状态和对象的行为逻辑有比较强的对应关系，即在不同状态时，对应的处理逻辑不一样；


![](https://raw.githubusercontent.com/retech-fe/image-hosting/main/img/2022/10/02/09-22-35-7f770bd51b24602ef1edc008777f615d-1358693242_5100-6e40df.jpeg)


在状态模式结构图中包含如下几个角色：

+  Context（环境类）：环境类又称为上下文类，它是拥有多种状态的对象。由于环境类的状态存在多样性且在不同状态下对象的行为有所不同，因此将状态独立出去形成单独的状态类。在环境类中维护一个抽象状态类 State 的实例，这个实例定义当前状态，在具体实现时，它是一个 State 子类的对象。

+ State（抽象状态类）：它用于定义一个接口以封装与环境类的一个特定状态相关的行为，在抽象状态类中声明了各种不同状态对应的方法，而在其子类中实现类这些方法，由于不同状态下对象的行为可能不同，因此在不同子类中方法的实现可能存在不同，相同的方法可以写在抽象状态类中。

+ ConcreteState（具体状态类）：它是抽象状态类的子类，每一个子类实现一个与环境类的一个状态相关的行为，每一个具体状态类对应环境的一个具体状态，不同的具体状态类其行为有所不同。

## 代码实现

![](https://raw.githubusercontent.com/retech-fe/image-hosting/main/img/2022/10/02/09-29-38-db32dc098e87db4649fa6427f54b559a-8aea4fca-62b1-4c5b-b41d-cad2211a2cbe-6d0823.png)

### 大多人的写法

```JavaScript
class Battery{
    constructor() {
        this.amount='high';
    }
    show() {
        if (this.amount == 'high') {
            console.log('绿色');
            this.amount='middle';
        }else if (this.amount == 'middle') {
            console.log('黄色');
            this.amount='low';
        }else{
            console.log('红色');
        }
    }
}
let battery=new Battery();
battery.show();
battery.show();
battery.show();
```

存在的问题

- show违反开放-封闭原则
- show方法(胖函数)逻辑太多太复杂
- 颜色状态切换不明显
- 过多的 if/else 让代码不可维护

### 优化一

```JavaScript
class SuccessState{
    show(){console.log('绿色');}
}
class WarningState{
    show(){console.log('黄色');}
}
class ErrorState{;'l  show(){console.log('红色');}
}
class WorstErrorState{
    show(){console.log('深红色');}
}
class Battery{
    constructor(){
        this.amount = 'high';
        this.state = new SuccessState();//绿色状态，满电的状态
    }
    show(){
        this.state.show();//把显示的逻辑委托给了状态对象
        //内部还要维护状态的变化 
        if(this.amount == 'high'){
            this.amount = 'middle';
            this.state = new WarningState();
        }else if(this.amount == 'middle'){
            this.amount = 'low';
            this.state = new ErrorState();
        }else if(this.amount == 'low'){
            this.amount = 'superlow';
            this.state = new WorstErrorState();
        }
    }
}
let battery = new Battery();
battery.show();
battery.show();
battery.show();
battery.show();
```

### 优化二

```JavaScript

class SuccessState {
  constructor(private battery: Battery) { }
  show() {
    console.log("绿色", this.battery.amount)
    this.battery.setState(new WarningState(this.battery))
  }
}
class WarningState {
  constructor(private battery: Battery) { }
  show() {
    console.log("黄色", this.battery.amount)
    this.battery.setState(new ErrorState(this.battery))
  }
}
class ErrorState {
  constructor(private battery: Battery) { }
  show() {
    console.log("红色", this.battery.amount)
    // this.battery.setState(new WorstErrorState(this.battery))
  }
}
// class WorstErrorState {
//   constructor(private battery: Battery) { }
//   show() {
//     console.log("深红色", this.battery.amount)
//   }
// }

class Battery {
  amount
  private state
  constructor() {
    this.amount = "high"
    this.state = new SuccessState(this) //绿色状态，满电的状态
  }
  setState(newState: any) {
    this.state = newState
  }
  show() {
    this.state.show() //把显示的逻辑委托给了状态对象
  }
}
let battery = new Battery()

battery.show()
battery.show()
battery.show()
battery.show()

```

## 应用场景

- 操作中含有庞大的多分支的条件语句，且这些分支依赖于该对象的状态，那么可以使用状态模式来将分支的处理分散到单独的状态类中；
- 对象的行为随着状态的改变而改变，那么可以考虑状态模式，来把状态和行为分离，虽然分离了，但是状态和行为是对应的，再通过改变状态调用状态对应的行为；

### Promise

```
class Promise {
  constructor(fn) {
    this.state = "initial" //先维护一下初始状态
    this.successes = []
    this.errors = []
    let resolve = (data) => {
      this.state = "fulfilled"
      this.successes.forEach((item) => item(data))
    }
    let reject = (error) => {
      this.state = "failed"
      this.errors.forEach((item) => item(error))
    }
    fn(resolve, reject)
  }
  then(success, error) {
    this.successes.push(success)
    this.errors.push(error)
  }
}
let p = new Promise(function (resolve, reject) {
  setTimeout(function () {
    let num = Math.random()
    if (num > 0.5) {
      resolve(num)
    } else {
      reject(num)
    }
  }, 500)
})
p.then(
  (data) => {
    console.log("成功", data)
  },
  (error) => {
    console.log("失败", error)
  }
)

```

### React导航

```
import { Button } from 'antd';
import { useState } from 'react';

const Banner = () => {
  const [state, setState] = useState<'show' | 'hide'>('show');
  // map映射
  const States = {
    show: function () {
      console.log('banner显示，点击可以关闭');
      //....
      setState('hide');
    },
    hide: function () {
      console.log('banner隐藏，点击可以打开');
      //.....
      setState('show');
    },-
  };

  const toggle = () => {
    States[state]();
  };
  return (
    <div>
      {state === 'show' && <nav>导航</nav>}
      <Button onClick={toggle}>{state === 'show' ? '隐藏' : '展示'}</Button>
    </div>
  );
};

export default Banner;
```

### 有限状态机
- 事物拥有多种状态，任一时间只会处于一种状态不会处于多种状态；
- 动作可以改变事物状态，一个动作可以通过条件判断，改变事物到不同的状态，但是不能同时指向多个状态，一个时间，就一个状态
- 状态总数是有限的；
- javascript-state-machine
  - form：当前行为从哪个状态来
  - to:当前行为执行完会过渡到哪个状态
  - name:当前行为的名字
- fsm.can(t) - return true 如果过渡方法t可以从当前状态触发
- fsm.cannot(t) - return true 如果当前状态下不能发生过渡方法t
- fsm.transitions() - 返回从当前状态可以过渡到的状态的列表
- fsm.allTransitions() - 返回所有过渡方法的列表
- fsm.allStates() - 返回状态机有的所有状态的列表
- onBefore 在特定动作TRANSITION前触发
- onLeaveState 离开任何一个状态的时候触发
- onEnter 进入一个特定的状态STATE时触发
- onLeave 在离开特定状态STATE时触发
- onTransition 在任何动作发生期间触发
- onEnterState 当进入任何状态时触发
- on onEnter的简写
- onAfterTransition 任何动作触发后触发
- onAfter 在特定动作TRANSITION后触发
- on onAfter的简写

```
// let StateMachine = require("javascript-state-machine")
class StateMachine {
  constructor(options) {
    //init定义初状态 transitions定义转换规则 methods定义监听 函数
    let { init = "", transitions = [], methods = {} } = options
    this.state = init
    transitions.forEach((transition) => {
      let { from, to, name } = transition
      this[name] = function () {
        if (this.state == from) {
          this.state = to
          let onMethod = "on" + name.slice(0, 1).toUpperCase() + name.slice(1) //onMelt
          methods[onMethod] && methods[onMethod]()
        }
      }
    })
  }
}
var fsm = new StateMachine({
  init: "solid",
  transitions: [
    { name: "melt", from: "solid", to: "liquid" },
    { name: "freeze", from: "liquid", to: "solid" },
    { name: "vaporize", from: "liquid", to: "gas" },
    { name: "condense", from: "gas", to: "liquid" },
  ],
  methods: {
    onMelt: function () {
      console.log("I melted")
    },
    onFreeze: function () {
      console.log("I froze")
    },
    onVaporize: function () {
      console.log("I vaporized")
    },
    onCondense: function () {
      console.log("I condensed")
    },
  },
})
fsm.melt()
fsm.freeze()

```

## 状态模式的优缺点

### 状态模式的优点：

- 结构相比之下清晰，避免了过多的 switch-case 或 if-else 语句的使用，避免了程序的复杂性提高系统的可维护性;
- 符合开闭原则，每个状态都是一个子类，增加状态只需增加新的状态类即可，修改状态也只需修改对应状态类就可以了；
- 封装性良好，状态的切换在类的内部实现，外部的调用无需知道类内部如何实现状态和行为的变换。

### 状态模式的缺点：
- 引入了多余的类，每个状态都有对应的类，导致系统中类的个数增加。

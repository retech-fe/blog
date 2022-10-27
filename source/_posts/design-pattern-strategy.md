---
title: 设计模式之策略模式
date: 2022-10-27 10:41:02
categories: design pattern
tags: design pattern
excerpt: 在这篇文章中，对设计模式中的策略模式从定义、类图、应用场景、优缺点以及和状态模式对比进行了详细的阐释。
author: hongxiang.gao
---


## 定义

策略模式 （Strategy Pattern）又称政策模式，其定义一系列的算法，把它们一个个封装起来，并且使它们可以互相替换。封装的策略算法一般是独立的，策略模式根据输入来调整采用哪个算法。

+ 关键是策略的实现和使用分离
+ 避免大量的if else 或 swith case

## 生活中的例子

### 例子

1. 现在电子产品种类繁多，尺寸多种多样，有时候你会忍不住想拆开看看里面啥样（想想小时候拆的玩具车还有遥控器），但是螺丝规格很多，螺丝刀尺寸也不少，如果每碰到一种规格就买一个螺丝刀，家里就得堆满螺丝刀了。所以现在人们都用多功能的螺丝刀套装，螺丝刀把只需要一个，碰到不同规格的螺丝只要换螺丝刀头就行了，很方便，体积也变小很多。

2. 一辆车的轮胎有很多规格，在泥泞路段开的多的时候可以用泥地胎，在雪地开得多可以用雪地胎，高速公路上开的多的时候使用高性能轮胎，针对不同使用场景更换不同的轮胎即可，不需更换整个车。

### 特点

- 螺丝刀头/轮胎（策略）之间相互独立，但又可以相互替换；
- 螺丝刀/车（封装上下文）可以根据需要的不同选用不同的策略；


## 案例

场景是这样的，某个电商网站希望举办一个活动，通过打折促销来销售库存物品，普通不打折，普通会员打9折，vip会员打8折

### 类图

![](https://raw.githubusercontent.com/retech-fe/image-hosting/main/img/2022/10/27/09-54-25-e297aa4b1e21a484e2f66c6e8cf081c7-20221027095425-e080b2.png)


### 代码

#### 大多数人的写法

``` TypeScript
type TCustomerType = 'normal' | 'member' | 'vip'
class Customer {
  constructor(private type: TCustomerType) {
    this.type = type
  }
  public pay(amount: number) {
    if (this.type == "member") {
      return amount * 0.9
    } else if (this.type == "vip") {
      return amount * 0.8
    } else {
      return amount
    }
  }
}
let c = new Customer("normal")
console.log(c.pay(100))
let c2 = new Customer("member")
console.log(c2.pay(100))
let c3 = new Customer("vip")
console.log(c2.pay(100))

```

#### 利用策略模式的优化

```
class Customer {
  constructor(public kind: Kind) {
  }
  public cost(amount: number) {
    return this.kind.discount(amount)
  }
}
abstract class Kind {
  abstract discount(amounr: number): number
}
class Normal extends Kind {
  discount(amount: number) {
    return amount
  }
}
class Member extends Kind {
  discount(amount: number) {
    return amount * 0.9
  }
}
class VIP extends Kind {
  discount(amount: number) {
    return amount * 0.8
  }
}
let c1 = new Customer(new Normal())
console.log(c1.cost(100))
c1.kind = new Member()
console.log(c1.cost(100))
c1.kind = new VIP()
console.log(c1.cost(100))

```

在前端也把算法封装在策略对象中，指定算法调用即可

```
class Customer {
  constructor() {
    this.kinds = {
      normal: function (price) {
        return price
      },
      member: function (price) {
        return price * 0.9
      },
      vip: function (price) {
        return price * 0.8
      },
    }
  }
  cost(kind, amount) {
    return this.kinds[kind](amount)
  }
}
let c = new Customer()
console.log(c.cost("normal", 100))
console.log(c.cost("member", 100))
console.log(c.cost("vip", 100))
```


## 应用场景

- 多个算法只在行为上稍有不同的场景，这时可以使用策略模式来动态选择算法；
- 算法需要自由切换的场景；
- 有时需要多重条件判断，那么可以使用策略模式来规避多重条件判断的情况；


### 表单校验 

```
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
</head>

<body>

  <form id="userForm">
    用户名 <input type="text" name="username"><br />
    密码 <input type="text" name="password"><br />
    手机号 <input type="text" name="mobile"><br />
    邮箱 <input type="text" name="email"><br />
    <input type="submit" value="提交">
  </form>
  <script>
    let form = document.getElementById('userForm');
    let validator = (function () {
      //这是一个规则的数组
      let rules = {
        notEmpty(val, msg) {
          if (val === '') {
            return msg;
          }
        },
        minLength(val, min, msg) {
          if (val === '' || val.length < min) {
            return msg;
          }
        },
        maxLength(val, max, msg) {
          if (val === '' || val.length > max) {
            return msg;
          }
        },
        isMobile(val, msg) {
          if (!/1\d{10}/.test(val)) {
            return msg;
          }
        }
      }
      function addRule(name, rule) {
        rules[name] = rule;
      }
      let checks = [];
      //增加校验的项目
      function add(element, rule) {
        checks.push(function () {
          let val = element.value;
          let name = rule.shift();
          rule.unshift(val);//['value','用户名不能为空']
          return rules[name] && rules[name].apply(element, rule);
        });
      }
      function start() {
        for (let i = 0; i < checks.length; i++) {
          let check = checks[i];
          let msg = check();
          if (msg) {
            return msg;
          }
        }
      }
      return {
        addRule,
        add,
        start
      }
    })();
    
    validator.addRule('isEmail', function (val, msg) {
      if (!/.*@.*/.test(val)) {
        return msg;
      }
    });
    form.onsubmit = function () {
      validator.add(form.username, ['notEmpty', '用户名不能为空']);
      validator.add(form.password, ['minLength', 6, '密码长度不能少于6位']);
      validator.add(form.password, ['maxLength', 8, '密码长度不能大于8位']);
      validator.add(form.mobile, ['isMobile', '必须输入合法的手机号']);
      validator.add(form.email, ['isEmail', '必须输入合法邮箱']);
      let msg = validator.start();
      if (msg) {
        alert(msg);
        return false;
      }
      return true;
    }
  </script>

</body>

</html>

```

## 策略模式的优缺点

策略模式将算法的实现和使用拆分，这个特点带来了很多优点：

- 策略之间相互独立，但策略可以自由切换，这个策略模式的特点给策略模式带来很多灵活性，也提高了策略的复用率；
- 如果不采用策略模式，那么在选策略时一般会采用多重的条件判断，采用策略模式可以避免多重条件判断，增加可维护性；
- 可扩展性好，策略可以很方便的进行扩展；

策略模式的缺点：

- 策略相互独立，因此一些复杂的算法逻辑无法共享，造成一些资源浪费；
- 如果用户想采用什么策略，必须了解策略的实现，因此所有策略都需向外暴露，这是违背迪米特法则/最少知识原则的，也增加了用户对策略对象的使用成本。


## 状态模式和策略模式异同

### 相同点：
- 策略模式和状态模式都有上下文，有策略或者状态类，上下文把这些请求委托给这些类来执行

### 不同点

- 状态模式： 重在强调对象内部状态的变化改变对象的行为，状态类之间是平行的，无法相互替换；
- 策略模式： 策略的选择由外部条件决定，策略可以动态的切换，策略之间是平等的，可以相互替换；
- 状态模式的状态类是平行的，意思是各个状态类封装的状态和对应的行为是相互独立、没有关联的，封装的业务逻辑可能差别很大毫无关联，相互之间不可替换。但是策略模式中的策略是平等的，是同一行为的不同描述或者实现，在同一个行为发生的时候，可以根据外部条件挑选任意一个实现来进行处理。
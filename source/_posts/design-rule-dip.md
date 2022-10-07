---
title: 设计原则之依赖倒置
date: 2022-10-07 12:06:02
categories: design pattern
tags: design pattern
excerpt: 在这篇文章中，对设计原则中的依赖倒置原则从概念、依赖、依赖的概念、依赖的方式等方方面进行了详细的阐释。
author: pengfei.zuo
---

## 一. 什么是依赖倒置原则

### 1.1 概念

依赖倒置原则(Dependence Inversion Principle, DIP), 其含义:

+ 高层模块不应该依赖低层模块，两者都应该依赖其抽象
+ 抽象不应该依赖细节, 细节应该依赖于抽象
+ 要针对接口编程，不要针对实现编程

### 1.2 什么是依赖

这里的依赖关系我们理解为UML关系中的依赖。简单的说就是A use B，那么A对B产生了依赖。具体请看下面的例子。

从上图中我们可以发现, 类A中的方法a()里面用到了类B, 其实这就是依赖关系, A依赖了B. 需要注意的是: 并不是说A中声明了B就叫依赖, 如果引用了但是没有真实调用方法, 那么叫做零耦合关系. 如下图:

![](https://raw.githubusercontent.com/retech-fe/image-hosting/main/img/2022/10/04/13-15-46-cb93cafee5c7141195ad93785db01aa3-20221004131546-ffe8c1.png)


### 1.3 依赖的关系种类

#### 1. 零耦合关系：如果两个类之间没有耦合关系，称之为零耦合

![](https://raw.githubusercontent.com/retech-fe/image-hosting/main/img/2022/10/04/13-36-53-f7cc21f00a0f687fb452df6cc3cc0ef4-20221004133652-221382.png)

#### 2. 直接耦合关系: 具体耦合发生在两个具体类（可实例化的）之间，经由一个类对另一个类的直接引用造成。

![](https://raw.githubusercontent.com/retech-fe/image-hosting/main/img/2022/10/04/13-38-35-08842da0a9f62f2fa2e4c401daec3732-20221004133834-e25ad8.png)

#### 3. 抽象耦合关系: 抽象耦合关系发生在一个具体类和一个抽象类（或者接口）之间，使两个必须发生关系的类之间存在最大的灵活性。

![](https://raw.githubusercontent.com/retech-fe/image-hosting/main/img/2022/10/07/09-29-32-a9e89ae9946cb4409c72cc49a5f60c45-20221007092931-fda3b3.png)


依赖倒转原则就是要针对接口编程，不要针对实现编程。这就是说，应当使用接口或者抽象类进行变量的类型声明，参数的类型声明，方法的返回类型说明，以及数据类型的转换等。


## 二. 依赖倒置的案例


### 2.1 初步设计方案

```
public class Benz {
    public void run() {
        System.out.println("奔驰跑起来了!");
    }
}

public class Driver {
    private String name;
    public Driver(String name) {
        this.name = name;
    }

    public void driver(Benz benz) {
        benz.run();
    }
}

public class CarTest {
    public static void main(String[] args) {
        Benz benz = new Benz();
        Driver driver = new Driver("张三");
        driver.driver(benz);
    }
}
```
有一个驾驶员张三可以驾驶奔驰汽车, 于是最开始我们思考, 会有一个驾驶员类, 有一个奔驰汽车类. 随着业务的发展, 我们发现, 驾驶员张三还可以驾驶宝马.

于是,我们定义一个BM类,

```
public class BM {
    public void run() {
        System.out.println("宝马跑起来了!");
    }
}
```
这时, 张三如果想要开宝马, 就要将宝马注册在他名下.

```
public class Driver {
    private String name;
    public Driver(String name) {
        this.name = name;
    }

    public void driver(Benz benz) {
        benz.run();
    }

    public void driver(BM bm) {
        bm.run();
    }

}

public class CarTest {
    public static void main(String[] args) {
        Benz benz = new Benz();
        BM bm = new BM();
        Driver driver = new Driver("张三");
        driver.driver(benz);
        driver.driver(bm);
    }
}
```
似乎这样就可以了, 但是这样有什么问题呢?

+ 如果张三有一天要开大众, 还要增加一个大众车类, 同时还得挂载司机名下.
+ 不是所有的人都要开奔驰, 开宝马. 开大众.

这就是面向实现编程的问题, 接下来我们就要考虑面向接口编程.

### 2.2 改进后的方案

```
public interface ICar {
    public void run();
}

public class Benz implements ICar{
    public void run() {
        System.out.println("奔驰跑起来了!");
    }
}

public class BM implements ICar{
    public void run() {
        System.out.println("宝马跑起来了!");
    }
}

public interface IDriver {
    public void driver(ICar car);
}

public class Driver implements IDriver{

    @Override
    public void driver(ICar car) {
        car.run();
    }
}

public class CarTest {
    public static void main(String[] args) {
        IDriver driver = new Driver();
        driver.driver(new Benz());
        driver.driver(new BM());
    }
}
```

修改后的代码, 提炼出来一个IDriver接口和ICar接口, 面向接口编程. IDriver的实现类驾驶员可以driver任何类型的汽车, 所以传入参数也是一个接口ICar. 任何类型的汽车, 都可以通过实现ICar接口注册为一种新的汽车类型. 当客户端调用的时候, 将对应的汽车传入就可以了.


## 三. 依赖的方式

### 3.1 依赖注入主要有三种方式：

#### 1. 构造注入，在构造的时候注入依赖

在类中通过构造函数声明依赖对象，按照依赖注入的说法，这种方式叫做构造函数注入，按照这种方式的注入，对IDriver和Driver进行修改。

```
public interface IDriver { //司机就会开车 public void drive(); 
}

public class Driver implements IDriver{
  private ICar car; //构造函数注入 
  public Driver(ICar _car){ this.car = _car; }
  //司机的主要职责就是驾驶汽车 
  public void drive(){ this.car.run(); } 

}
```
#### 2. Setter方法注入

在抽象中设置Setter方法声明依赖关系，依照依赖注入的说法，这是Setter依赖注入，按照这种方式的注入，对IDriver和Driver进行修改：

```
public interface IDriver { //车辆型号 
  public void setCar(ICar car);
 //是司机就应该会驾驶汽车
  public void drive(); 
}

public class Driver implements IDriver{
  private ICar car; public void setCar(ICar car){ this.car = car; }

 //司机的主要职责就是驾驶汽车
 public void drive(){ this.car.run();
  } 

}

```
#### 3. 接口声明依赖对象
在接口的方法中声明依赖对象，未修改的IDriver和Driver就采用了接口声明依赖的方式，该方法也叫做接口注入。

```
public interface IDriver { 
 //老司机，会开车
  public void drive(ICar car); 
}

public class Driver implements IDriver{ //司机的主要职责就是驾驶汽车  public void drive(ICar car){ car.run(); } 
}
```

### 3.2 依赖倒置原则在设计模式中的体现

+ 简单工厂设计模式, 使用的是接口方法中注入
+ 策略设计模式: 在构造函数中注入.
---
title: 设计原则之迪米特法则
date: 2022-10-07 21:57:02
categories: design pattern
tags: design pattern
excerpt: 在这篇文章中，对设计原则中的迪米特法则从定义、实践、注意事项等方面进行了详细的阐释。
author: pengfei.zuo
---

## 一. 什么是迪米特法则

迪米特法则(Law of Demeter )又叫做最少知识原则，也就是说，一个对象应当对其他对象尽可能少的了解。不和陌生人说话。英文简写为: LoD。

迪米特法则的目的在于降低类之间的耦合。由于每个类尽量减少对其他类的依赖，因此，很容易使得系统的功能模块功能独立，相互之间不存在（或很少有）依赖关系。

迪米特法则不希望类之间建立直接的联系。如果真的有需要建立联系，也希望能通过它的友元类来转达。因此，应用迪米特法则有可能造成的一个后果就是：系统中存在大量的中介类，这些类之所以存在完全是为了传递类之间的相互调用关系——这在一定程度上增加了系统的复杂度。


## 二. 为什么要遵守迪米特法则?

在面向对象编程中有一些众所周知的抽象概念，比如封装、内聚和耦合，理论上可以用来生成清晰的设计和良好的代码。虽然这些都是非常重要的概念，但它们不够实用，不能直接用于开发环境，这些概念是比较主观的，非常依赖于使用人的经验和知识。对于其他概念，如单一责任原则、开闭原则等，情况也是一样的。迪米特法则的独特之处在于它简洁而准确的定义，它允许在编写代码时直接应用，几乎自动地应用了适当的封装、低内聚和松耦合。

## 三. 迪米特法则的广狭定义

### 3.1. 狭义的迪米特法则

如果两个类不必彼此直接通信，那么这两个类就不应当发生直接的相互作用。如果其中的一个类需要调用另一个类的某一个方法的话，可以通过第三者转发这个调用。

朋友圈的确定“朋友”条件：

 1）当前对象本身（this）

 2）以参数形式传入到当前对象方法中的对象

 方法入参是一个对象, 这是这个对象和当前类是朋友

 3）当前对象的实例变量直接引用的对象

 定一个一个类, 里面的属性引用了其他对象, 那么这个对象的实例和当前实例是朋友

 4）当前对象的实例变量如果是一个聚集，那么聚集中的元素也都是朋友

 如果属性是一个对象, 那么属性和对象里的元素都是朋友

 5）当前对象所创建的对象

任何一个对象，如果满足上面的条件之一，就是当前对象的“朋友”；否则就是“陌生人”


狭义的迪米特法则的缺点：

在系统里造出大量的小方法，这些方法仅仅是传递间接的调用，与系统的业务逻辑无关。
遵循类之间的迪米特法则会是一个系统的局部设计简化，因为每一个局部都不会和远距离的对象有直接的关联。但是，这也会造成系统的不同模块之间的通信效率降低，也会使系统的不同模块之间不容易协调。


### 3.2. 广义的迪米特法则在类的设计上的体现

  + 优先考虑将一个类设置成不变类。
  + 尽量降低一个类的访问权限。
  + 尽量降低成员的访问权限。

## 四. 迪米特法则在设计模式中的应用

设计模式的门面模式（Facade）和中介模式（Mediator），都是迪米特法则的应用

下面我们已经租房为例, 来研究迪米特法则。

通常 客户要找房子住, 我们就直接建一个房子类, 建一个客户类, 客户找房子即可。

```
public interface IHouse {
    // 住房子
    public void Housing();
}

public class House implements IHouse{

    @Override
    public void Housing() {
        System.out.println("住房子");
    }
}


public class Customer {
    public String name;

    public void findHourse(IHouse house) {
        house.Housing();
    }
}
```
客户找房子住, 逻辑很简单, 这样是ok的。虽然违背了迪米特法则, 但符合业务逻辑也说得通。

但是, 通常我们找房子, 不是一下子就能找到的, 我们要找很多家, 这就很费劲, 那不如交给中介。

中介有很多房源, 房东吧房子给了中介, 不需要关心租户是谁, 租户将找房的事交给房东, 他也不用管房东是谁, 而且租户+房东都很省事。

```
/**
 * 房子
 */
public interface IHouse {
    // 住房子
    public void Housing();
}

public class House implements IHouse{

    @Override
    public void Housing() {
        System.out.println("住房子");
    }
}

public interface ICustomer {

    void findHourse(IHouse house) ;
}

public class Customer implements ICustomer {

    public void findHourse(IHouse house) {
        house.Housing();
    }
}

/**
 * 中介
 */
public class Intermediary {
    // 找房子
    public IHouse findHouse(ICustomer customer){
        // 帮租户找房子
        return null;
    }
}
```

房子,客户是相互独立的, 彼此之间没有引用。他们之间建立关系是通过中介。 也就是, 客户找中介租房子, 房东吧房子交给租户, 最后中介将找好的房子给到客户。客户和房东彼此隔离, 符合迪米特法则。

## 五. 迪米特法则实践

那么在实践中如何做到一个对象应该对其他对象有最少的了解呢？

如果我们把一个对象看作是一个人，那么要实现“一个人应该对其他人有最少的了解”，做到两点就足够了：

 + 只和直接的朋友交流；

 + 减少对朋友的了解。

 下面就详细说说如何做到这两点。
 
 1. 只和直接的朋友交流
 
迪米特法则还有一个英文解释是：talk only to your immediate friends（只和直接的朋友交流）。

什么是朋友呢？

每个对象都必然会与其他的对象有耦合关系，两个对象之间的耦合就会成为朋友关系。那么什么又是直接的朋友呢？出现在成员变量、方法的输入输出参数中的类就是直接的朋友。迪米特法则要求只和直接的朋友通信。

 > 注意：
 
 > 只出现在方法体内部的类就不是直接的朋友，如果一个类和不是直接的朋友进行交流，就属于违反迪米特法则。
 
我们举一个例子说明什么是朋友，什么是直接的朋友。很简单的例子：老师让班长清点全班同学的人数。这个例子中总共有三个类：老师Teacher、班长GroupLeader和学生Student。

 ```
 public interface ITeacher {
    void command(IGroupLeader groupLeader);
}

public class Teacher implements ITeacher{
    @Override
    public void command(IGroupLeader groupLeader) {
        // 全班同学
        List<Student> allStudent = new ArrayList<>();
        allStudent.add(new Student());
        allStudent.add(new Student());
        allStudent.add(new Student());
        allStudent.add(new Student());
        allStudent.add(new Student());
        // 班长清点人数
        groupLeader.count(allStudent);

    }
}

**
 * 班长类
 */
public interface IGroupLeader {

    // 班长清点人数
    void count(List<Student> students);
}

/**
 * 班长类
 */
public class GroupLeader implements IGroupLeader{
    /**
     * 班长清点人数
     * @param students
     */
    @Override
    public void count(List<Student> students) {
        // 班长清点人数
        System.out.println("上课的学生人数是: " + students.size());
    }
}

/**
 * 学生类
 */
public interface IStudent {
}

/**
 * 学生类
 */
public class Student implements IStudent {


}

/**
 * 客户端
 */
public class Client {
    public static void main(String[] args) {
        // 老师类
        ITeacher wangTeacher = new Teacher();

        // 班长
        IGroupLeader zhangBanzhang = new GroupLeader();
        wangTeacher.command(zhangBanzhang);
    }
}
 ```
 
 运行结果:

```
 上课的学生人数是: 5
```

在这个例子中，我们的Teacher有几个朋友？两个，一个是GroupLeader，它是Teacher的command()方法的入参；另一个是Student，因为在Teacher的command()方法体中使用了Student。

那么Teacher有几个是直接的朋友？按照直接的朋友的定义

> 出现在成员变量、方法的输入输出参数中的类就是直接的朋友


只有GroupLeader是Teacher的直接的朋友。

Teacher在command()方法中创建了Student的数组，和非直接的朋友Student发生了交流，所以，上述例子违反了迪米特法则。方法是类的一个行为，类竟然不知道自己的行为与其他的类产生了依赖关系，这是不允许的，严重违反了迪米特法则！

为了使上述例子符合迪米特法则，我们可以做如下修改：

```
public interface ITeacher {
    void command(IGroupLeader groupLeader);
}

public class Teacher implements ITeacher {
    @Override
    public void command(IGroupLeader groupLeader) {
        // 班长清点人数
        groupLeader.count();
    }
}

/**
 * 班长类
 */
public interface IGroupLeader {
    // 班长清点人数
    void count();
}

/**
 * 班长类
 */
public class GroupLeader implements IGroupLeader {

    private List<Student> students;

    public GroupLeader(List<Student> students) {
        this.students = students;
    }

    /**
     * 班长清点人数
     */
    @Override
    public void count() {
        // 班长清点人数
        System.out.println("上课的学生人数是: " + students.size());
    }
}


/**
 * 学生类
 */
public interface IStudent {
}

/**
 * 学生类
 */
public class Student implements IStudent {


}


/**
 * 客户端
 */
public class Client {
    public static void main(String[] args) {
        // 老师类
        ITeacher wangTeacher = new Teacher();

        List<Student> allStudent = new ArrayList(10);
        allStudent.add(new Student());
        allStudent.add(new Student());
        allStudent.add(new Student());
        allStudent.add(new Student());

        // 班长
        IGroupLeader zhangBanzhang = new GroupLeader(allStudent);
        wangTeacher.command(zhangBanzhang);
    }
}
```

运行结果:

```
 上课的学生人数是: 4
```

这样修改后，每个类都只和直接的朋友交流，有效减少了类之间的耦合

2. 减少对朋友的了解

如何减少对朋友的了解？如果你的朋友是个话痨加大喇叭，那就算你不主动去问他，他也会在你面前说个不停，把他所有的经历都讲给你听。所以，要减少对朋友的了解，请换一个内敛一点的朋友吧～换作在一个类中，就是尽量减少一个类对外暴露的方法。

举一个简单的例子说明一个类暴露方法过多的情况。一个人用咖啡机煮咖啡的过程，例子中只有两个类，一个是人，一个是咖啡机。

首先是咖啡机类CoffeeMachine，咖啡机制作咖啡只需要三个方法：1.加咖啡豆；2.加水；3.制作咖啡：


```
/**
 * 咖啡机抽象接口
 */
public interface ICoffeeMachine {

    //加咖啡豆
    void addCoffeeBean();

    //加水
    void addWater();

    //制作咖啡
    void makeCoffee();
}


/**
 * 咖啡机实现类
 */
public class CoffeeMachine implements ICoffeeMachine{

    //加咖啡豆
    public void addCoffeeBean() {
        System.out.println("放咖啡豆");
    }

    //加水
    public void addWater() {
        System.out.println("加水");
    }

    //制作咖啡
    public void makeCoffee() {
        System.out.println("制作咖啡");
    }
}


/**
 * 人, 制作咖啡
 */
public interface IMan {
    /**
     * 制作咖啡
     */
    void makeCoffee();
}

/**
 * 人制作咖啡
 */
public class Man implements IMan {
    private ICoffeeMachine coffeeMachine;

    public Man(ICoffeeMachine coffeeMachine) {
        this.coffeeMachine = coffeeMachine;
    }

    /**
     * 制作咖啡
     */
    public void makeCoffee() {
        coffeeMachine.addWater();
        coffeeMachine.addCoffeeBean();
        coffeeMachine.makeCoffee();
    }
}

/**
 * 客户端
 */
public class Client {
    public static void main(String[] args) {
        ICoffeeMachine coffeeMachine = new CoffeeMachine();

        IMan man = new Man(coffeeMachine);
        man.makeCoffee();

    }
}
```

运行结果:

```

 加水

 放咖啡豆

 制作咖啡
```

在这个例子中，CoffeeMachine是Man的直接好友，但问题是Man对CoffeeMachine了解的太多了，其实人根本不关心咖啡机具体制作咖啡的过程。所以我们可以作如下优化：

优化后的咖啡机类，只暴露一个work方法，把制作咖啡的三个具体的方法addCoffeeBean、addWater、makeCoffee设为私有:

```
/**
 * 咖啡机抽象接口
 */
public interface ICoffeeMachine {

    //咖啡机工作
    void work();

}

/**
 * 咖啡机实现类
 */
public class CoffeeMachine implements ICoffeeMachine {

    //加咖啡豆
    public void addCoffeeBean() {
        System.out.println("放咖啡豆");
    }

    //加水
    public void addWater() {
        System.out.println("加水");
    }

    //制作咖啡
    public void makeCoffee() {
        System.out.println("制作咖啡");
    }

    @Override
    public void work() {
        addCoffeeBean();
        addWater();
        makeCoffee();
    }
}

/**
 * 人, 制作咖啡
 */
public interface IMan {
    /**
     * 制作咖啡
     */
    void makeCoffee();
}


/**
 * 人制作咖啡
 */
public class Man implements IMan {
    private ICoffeeMachine coffeeMachine;

    public Man(ICoffeeMachine coffeeMachine) {
        this.coffeeMachine = coffeeMachine;
    }

    /**
     * 制作咖啡
     */
    public void makeCoffee() {
        coffeeMachine.work();
    }
}

/**
 * 客户端
 */
public class Client {
    public static void main(String[] args) {
        ICoffeeMachine coffeeMachine = new CoffeeMachine();

        IMan man = new Man(coffeeMachine);
        man.makeCoffee();

    }
}
```

这样修改后，通过减少CoffeeMachine对外暴露的方法，减少Man对CoffeeMachine的了解，从而降低了它们之间的耦合。

在实践中，只要做到只和直接的朋友交流和减少对朋友的了解，就能满足迪米特法则。因此我们不难想象，迪米特法则的目的，是把我们的类变成一个个“肥宅”。“肥”在于一个类对外暴露的方法可能很少，但是它内部的实现可能非常复杂（这个解释有点牵强~）。“宅”在于它只和直接的朋友交流。在现实生活中“肥宅”是个贬义词，在日本“肥宅”已经成为社会问题。但是在程序中，一个“肥宅”的类却是优秀类的典范

## 六. 注意事项

+ 第一：在类的划分上，应当创建弱耦合的类，类与类之间的耦合越弱，就越有利于实现可复用的目标。
+ 第二：在类的结构设计上，每个类都应该降低成员的访问权限。
+ 第三：在类的设计上，只要有可能，一个类应当设计成不变的类。
+ 第四：在对其他类的引用上，一个对象对其他类的对象的引用应该降到最低。
+ 第五：尽量限制局部变量的有效范围，降低类的访问权限。


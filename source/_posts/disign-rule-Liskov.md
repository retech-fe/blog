---
title: 设计原则之里氏替换原则
date: 2022-10-04 10:35:02
categories: design pattern
tags: design pattern
excerpt: 在这篇文章中，对设计原则中的里氏替换原则从定义、目的、使用规则等方面进行了详细的阐释。
author: pengfei.zuo
---

## 一、里氏替换的定义 

### 1.1 里式替换原则定义

里氏替换原则（Liskov Substitution Principle, LSP）也叫里氏代换原则；里氏替换原则最早是在1988年，由麻省理工学院的一位姓里的女士（Barbara Liskov）提出来的。

#### 定义一：

> If S is a subtype of T, then objects of type T may be replaced with objects of type S, without breaking the program。

 > 如果S是T的子类，则T的对象可以替换为S的对象，而不会破坏程序。
 
 
#### 定义二： 
 
 > Functions that use pointers of references to base classes must be able to use objects of derived classes without knowing it。
 
 > 所有引用其父类对象方法的地方，都可以透明的替换为其子类对象。


### 1.2 里氏替换原则的两个含义：

+  里氏替换原则是针对继承而言的，继承是为了实现代码重用，也就是为了共享方法，那么共享的父类方法就应该保持不变，不能被子类重新定义。子类只能通过新添加方法来扩展功能，父类和子类都可以实例化，而子类继承的方法和父类是一样的，父类调用方法的地方，子类也可以调用同一个继承得来的，逻辑和父类一致的方法，这时用子类对象将父类对象替换掉时，因为逻辑一致，所以相安无事。

+  如果继承的目的是为了多态，而多态的前提就是子类覆盖并重新定义父类的方法，为了符合LSP，我们应该将父类定义为抽象类，并定义抽象方法，让子类重新定义这些方法，当父类是抽象类时，父类就是不能实例化，所以也不存在可实例化的父类对象在程序里。也就不存在子类替换父类实例（根本不存在父类实例了）时逻辑不一致的可能。

不符合LSP的最常见的情况是，父类和子类都是可实例化的非抽象类，且父类的方法被子类重新定义，这一类的实现继承会造成父类和子类间的强耦合，也就是实际上并不相关的属性和方法牵强附会在一起，不利于程序扩展和维护。


## 二、里氏替换的目的

采用里氏替换原则就是为了减少继承带来的缺点，增强程序的健壮性，版本升级时也可以保持良好的兼容性。即使增加子类，原有的子类也可以继续运行。


## 三、里式替换原则与继承多态之间的关系

里式替换原则和继承多态有关系, 但是他俩并不是一回事. 我们来看看下面的案例

```
public class Cache {
    public void set(String key, String value) {

    }
}

public class Redis extends Cache {
    @Override
    public void set(String key, String value) {

    }
}


public class Memcache extends Cache {
    @Override
    public void set(String key, String value) {

    }
}

public class CacheTest {
    public static void main(String[] args) {
        // 父类对象都可以接收子类对象
        Cache cache = new Cache();
        cache.set("key123", "key123");

        cache = new Redis();
        cache.set("key123", "key123");

        cache = new Memcache();
        cache.set("key123", "key123");
    }
}
```
通过上面的例子, 可以看出Cache是父类, Redis 和 Memcache是子类, 他们继承自Cache. 这是继承和多态的思想. 而且这两个子类目前为止也都符合里式替换原则.可以替换父类出现的任何位置，并且原来代码的逻辑行为不变且正确性也没有被破坏。

看最后的CacheTest类, 我们使用父类的cache可以接收任何一种类型的缓存对象, 包括父类和子类。

但如果我们对Redis中的set方法做了长度校验

```
public class Redis extends Cache{
    @Override
    public void set(String key, String value) {
        if (key == null || key.length() < 10 || key.length() > 100) {
            System.out.println("key的长度不符合要求");
            throw new IllegalArgumentException(key的长度不符合要求);
        }
    }
}

public class CacheTest {
    public static void main(String[] args) {
        // 父类对象都可以接收子类对象
        Cache cache = new Cache();
        cache.set("key123", "key123");

        cache = new Redis();
        cache.set("key123", "key123");
    }
}
```

如上情况, 如果我们使用父类对象时替换成子类对象, 那么就会抛出异常. 程序的逻辑行为就发生了变化，虽然改造之后的代码仍然可以通过子类来替换父类 ，但是，从设计思路上来讲，Redis子类的设计是不符合里式替换原则的。

继承和多态是面向对象语言所提供的一种语法，是代码实现的思路，而里式替换则是一种思想，一种设计原则，是用来指导继承关系中子类该如何设计的，子类的设计要保证在替换父类的时候，不改变原有程序的逻辑以及不破坏原有程序的正确性。

## 四、里式替换的规则

里式替换原则的核心就是“约定”，父类与子类的约定。里氏替换原则要求子类在进行设计的时候要遵守父类的一些行为约定。这里的行为约定包括：函数所要实现的功能，对输入、输出、异常的约定，甚至包括注释中一些特殊说明等。

### 4.1 子类方法不能违背父类方法对输入输出异常的约定

#### 1. 前置条件不能被加强

前置条件即输入参数是不能被加强的，就像上面Cache的示例，Redis子类对输入参数Key的要求进行了加强，此时在调用处替换父类对象为子类对象就可能引发异常。

也就是说，子类对输入的数据的校验比父类更加严格，那子类的设计就违背了里式替换原则。

#### 2. 后置条件不能被削弱

后置条件即输出，假设我们的父类方法约定输出参数要大于0，调用父类方法的程序根据约定对输出参数进行了大于0的验证。而子类在实现的时候却输出了小于等于0的值。此时子类的涉及就违背了里氏替换原则

### 3. 不能违背对异常的约定
在父类中，某个函数约定，只会抛出 ArgumentNullException 异常， 那子类的设计实现中只允许抛出 ArgumentNullException 异常，任何其他异常的抛出，都会导致子类违背里式替换原则。


### 4.2 子类方法不能违背父类方法定义的功能

```
public class Product {
    private BigDecimal amount;
    private Calendar createTime;
 
    public BigDecimal getAmount() {
        return amount;
    }
    public void setAmount(BigDecimal amount) {
        this.amount = amount;
    }
 
    public Calendar getCreateTime() {
        return createTime;
    }
    public void setCreateTime(Calendar createTime) {
        this.createTime = createTime;
    }
}
 
public class ProductSort extends Sort<Product> {
 
    public void sortByAmount(List<Product> list) {
        //根据时间进行排序
        list.sort((h1, h2)->h1.getCreateTime().compareTo(h2.getCreateTime()));
    }
}
```

父类中提供的 sortByAmount() 排序函数，是按照金额从小到大来进行排序的，而子类重写这个 sortByAmount() 排序函数之后，却是是按照创建日期来进行排序的。那子类的设计就违背里式替换原则。

实际上对于如何验证子类设计是否符合里氏替换原则其实有一个小技巧，那就是你可以使用父类的单测来运行子类的代码，如果不可以正常运行，那么你就要考虑一下自己的设计是否合理了！


### 4.3 子类必须完全实现父类的抽象方法

如果你设计的子类不能完全实现父类的抽象方法那么你的设计就不满足里式替换原则。

```
// 定义抽象类枪
public abstract class AbstractGun{
    // 射击
    public abstract void shoot();
    
    // 杀人
    public abstract void kill();
}
```
比如我们定义了一个抽象的枪类，可以射击和杀人。无论是步枪还是手枪都可以射击和杀人，我们可以定义子类来继承父类

```
// 定义手枪，步枪，机枪
public class Handgun extends AbstractGun{   
    public void shoot(){  
         // 手枪射击
    }
    
    public void kill(){    
        // 手枪杀人
    }
}
public class Rifle extends AbstractGun{
    public void shoot(){
         // 步枪射击
    }
    
    public void kill(){    
         // 步枪杀人
    }
}
```

但是如果我们在这个继承体系内加入一个玩具枪，就会有问题了，因为玩具枪只能射击，不能杀人。但是很多人写代码经常会这么写。

```
public class ToyGun extends AbstractGun{
    public void shoot(){
        // 玩具枪射击
    }
    
    public void kill(){ 
        // 因为玩具枪不能杀人，就返回空，或者直接throw一个异常出去
        throw new Exception("我是个玩具枪，惊不惊喜，意不意外，刺不刺激？");
    }
}
```

这时，我们如果把使用父类对象的地方替换为子类对象，显然是会有问题的(士兵上战场结果发现自己拿的是个玩具)。

而这种情况不仅仅不满足里氏替换原则，也不满足接口隔离原则，对于这种场景可以通过 ** 接口隔离+委托** 的方式来解决。

## 四. 里氏替换原则的作用

+ 里氏替换原则是实现开闭原则的重要方式之一。
+ 它克服了继承中重写父类造成的可复用性变差的缺点。
+ 它是动作正确性的保证。即类的扩展不会给已有的系统引入新的错误，降低了代码出错的可能性。
+ 加强程序的健壮性，同时变更时可以做到非常好的兼容性，提高程序的维护性、可扩展性，降低需求变更时引入的风险。

尽量不要从可实例化的父类中继承，而是要使用基于抽象类和接口的继承。

## 五. 里氏替换原则的实现方法

里氏替换原则通俗来讲就是：子类可以扩展父类的功能，但不能改变父类原有的功能。也就是说：子类继承父类时，除添加新的方法完成新增功能外，尽量不要重写父类的方法。

根据上述理解，对里氏替换原则的定义可以总结如下：

+ 子类可以实现父类的抽象方法，但不能覆盖父类的非抽象方法
+ 子类中可以增加自己特有的方法
+ 当子类的方法重载父类的方法时，方法的前置条件（即方法的输入参数）要比父类的方法更宽松
+ 当子类的方法实现父类的方法时（重写/重载或实现抽象方法），方法的后置条件（即方法的的输出/返回值）要比父类的方法更严格或相等

通过重写父类的方法来完成新的功能写起来虽然简单，但是整个继承体系的可复用性会比较差，特别是运用多态比较频繁时，程序运行出错的概率会非常大。

如果程序违背了里氏替换原则，则继承类的对象在基类出现的地方会出现运行错误。这时其修正方法是：取消原来的继承关系，重新设计它们之间的关系。

关于里氏替换原则的例子，最有名的是“正方形不是长方形”。当然，生活中也有很多类似的例子，例如，企鹅、鸵鸟和几维鸟从生物学的角度来划分，它们属于鸟类；但从类的继承关系来看，由于它们不能继承“鸟”会飞的功能，所以它们不能定义成“鸟”的子类。同样，由于“气球鱼”不会游泳，所以不能定义成“鱼”的子类；“玩具炮”炸不了敌人，所以不能定义成“炮”的子类等。

## 六. 案例分析

### 案例一: 两数相减

当使用继承时，遵循里氏替换原则。类B继承类A时，除添加新的方法完成新增功能P2外，尽量不要重写父类A的方法，也尽量不要重载父类A的方法。

继承包含这样一层含义：父类中凡是已经实现好的方法（相对于抽象方法而言），实际上是在设定一系列的规范和契约，虽然它不强制要求所有的子类必须遵从这些契约，但是如果子类对这些非抽象方法任意修改，就会对整个继承体系造成破坏。而里氏替换原则就是表达了这一层含义。

> 继承作为面向对象三大特性之一，在给程序设计带来巨大便利的同时，也带来了弊端。比如使用继承会给程序带来侵入性，程序的可移植性降低，增加了对象间的耦合性，如果一个类被其他的类所继承，则当这个类需要修改时，必须考虑到所有的子类，并且父类修改后，所有涉及到子类的功能都有可能会产生故障。


```
class A{
	public int func1(int a, int b){
		return a-b;
	}
}
 
public class Client{
	public static void main(String[] args){
		A a = new A();
		System.out.println("100-50="+a.func1(100, 50));
		System.out.println("100-80="+a.func1(100, 80));
	}
}
```

运行结果：

```
100-50=50
100-80=20
```

后来，我们需要增加一个新的功能：完成两数相加，然后再与100求和，由类B来负责。即类B需要完成两个功能：

+ 两数相减。
+ 两数相加，然后再加100。

由于类A已经实现了第一个功能，所以类B继承类A后，只需要再完成第二个功能就可以了，代码如下：

```
class B extends A{
	public int func1(int a, int b){
		return a+b;
	}
	
	public int func2(int a, int b){
		return func1(a,b)+100;
	}
}
 
public class Client{
	public static void main(String[] args){
		B b = new B();
		System.out.println("100-50="+b.func1(100, 50));
		System.out.println("100-80="+b.func1(100, 80));
		System.out.println("100+20+100="+b.func2(100, 20));
	}
}
```

类B完成后，运行结果：

```
100-50=150
100-80=180
100+20+100=220
```

我们发现原本运行正常的相减功能发生了错误。原因就是类B在给方法起名时无意中重写了父类的方法，造成所有运行相减功能的代码全部调用了类B重写后的方法，造成原本运行正常的功能出现了错误。在本例中，引用基类A完成的功能，换成子类B之后，发生了异常。在实际编程中，我们常常会通过重写父类的方法来完成新的功能，这样写起来虽然简单，但是整个继承体系的可复用性会比较差，特别是运用多态比较频繁时，程序运行出错的几率非常大。如果非要重写父类的方法，比较通用的做法是：原来的父类和子类都继承一个更通俗的基类，原有的继承关系去掉，采用依赖、聚合，组合等关系代替。


### 案例二: "几维鸟不是鸟"

需求分析: 鸟通常都是会飞的, 比如燕子每小时120千米, 但是新西兰的几维鸟由于翅膀退化不会飞. 假如要设计一个实例，计算这两种鸟飞行 300 千米要花费的时间。显然，拿燕子来测试这段代码，结果正确，能计算出所需要的时间；但拿几维鸟来测试，结果会发生“除零异常”或是“无穷大”，明显不符合预期，其类图如图 1 所示。


![](https://raw.githubusercontent.com/retech-fe/image-hosting/main/img/2022/10/04/10-17-00-2635dfd2beed32cc20311e68bdc18db4-20221004101659-1be3aa.png)

源码如下:

```
/**
 * 鸟
 */
public class Bird {
    // 飞行的速度
    private double flySpeed;

    public void setFlySpeed(double flySpeed) {
        this.flySpeed = flySpeed;
    }

    public double getFlyTime(double distance) {
        return distance/flySpeed;
    }
}

/**
 * 燕子
 */
public class Swallow extends Bird{
}

/**
 * 几维鸟
 */
public class Kiwi extends Bird {
    @Override
    public void setFlySpeed(double flySpeed) {
        flySpeed = 0;
    }
}

/**
  * 测试飞行耗费时间
  */
public class BirdTest {
    public static void main(String[] args) {
        Bird bird1 = new Swallow();
        Bird bird2 = new Kiwi();
        bird1.setFlySpeed(120);
        bird2.setFlySpeed(120);
        System.out.println("如果飞行300公里：");
        try {
            System.out.println("燕子花费" + bird1.getFlyTime(300) + "小时.");
            System.out.println("几维花费" + bird2.getFlyTime(300) + "小时。");
        } catch (Exception err) {
            System.out.println("发生错误了!");
        }
    }
}
```
运行结果

```
 如果飞行300公里：
 燕子花费2.5小时.
 几维花费Infinity小时。
```
程序运行错误的原因是：几维鸟类重写了鸟类的 setSpeed(double speed) 方法，这违背了里氏替换原则。正确的做法是：取消几维鸟原来的继承关系，定义鸟和几维鸟的更一般的父类，如动物类，它们都有奔跑的能力。几维鸟的飞行速度虽然为 0，但奔跑速度不为 0，可以计算出其奔跑 300 千米所要花费的时间。其类图如图 2 所示。

![](https://raw.githubusercontent.com/retech-fe/image-hosting/main/img/2022/10/04/10-18-27-29b90e34495cb07c425c3fd0961c5d5b-20221004101826-20631d.png)


源代码实现如下

```
/**
 * 动物
 */
public class Animal {
    private double runSpeed;

    public double getRunTime(double distance) {
        return distance/runSpeed;
    }

    public void setRunSpeed(double runSpeed) {
        this.runSpeed = runSpeed;
    }
}


/**
 * 鸟
 */
public class Bird {
    // 飞行的速度
    private double flySpeed;

    public void setFlySpeed(double flySpeed) {
        this.flySpeed = flySpeed;
    }

    public double getFlyTime(double distance) {
        return distance/flySpeed;
    }
}

/**
 * 燕子
 */
public class Swallow extends Bird {
}

/**
 * 几维鸟
 */
public class Kiwi extends Animal {
    @Override
    public void setRunSpeed(double runSpeed) {
        super.setRunSpeed(runSpeed);
    }
}

/**
  * 测试飞行耗费时间
  */
public class BirdTest {
    public static void main(String[] args) {
        Bird bird1 = new Swallow();
        Animal bird2 = new Kiwi();
        bird1.setFlySpeed(120);
        bird2.setRunSpeed(110);
        System.out.println("如果飞行300公里：");
        try {
            System.out.println("燕子花费" + bird1.getFlyTime(300) + "小时.");
            System.out.println("几维鸟花费" + bird2.getRunTime(300) + "小时。");
        } catch (Exception err) {
            System.out.println("发生错误了!");
        }
    }
}
```
运行结果

```
如果飞行300公里：
 燕子花费2.5小时.
 几维鸟花费2.727272727272727小时。
```

## 总结

面向对象的编程思想中提供了继承和多态是我们可以很好的实现代码的复用性和可扩展性，但继承并非没有缺点，因为继承的本身就是具有侵入性的，如果使用不当就会大大增加代码的耦合性，而降低代码的灵活性，增加我们的维护成本，然而在实际使用过程中却往往会出现滥用继承的现象，而里式替换原则可以很好的帮助我们在继承关系中进行父子类的设计。


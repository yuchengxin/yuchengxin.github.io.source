---
title: <font color=#0099ff size=6 face="微软雅黑">Head First设计模式-策略模式</font>
date: 2017-05-23 08:59:26
categories: java读书笔记
tags: [java,设计模式,策略模式]
---
**设计原则：**
=========

 - 找出应用中可能需要变化之处，把他们独立出来，不要和那些不需要变化的代码混在一起。把会变化的部分取出并封装起来，好让其他部分不会受到影响。
 - 针对接口编程，而不是针对实现编程。“针对接口编程”的真正意思是针对超类编程。即：变量的申明类应该是超类型，通常是一个抽象类或者一个接口，如此，只要是具体实现此超类的类所产生的对象，都可以指定给这个变量。这也意味着，声明类时不用理会以后执行时的真正对象类型。
 - 多用组合，少用继承

**举例：**
===
世界上有各种鸭子，它们外观不同，但是都会游泳，也都会叫。用传统的面向对象的思路来设计，首先有一个Duck的抽象超类，并让各种鸭子继承这个超类。

| Duck |
|:------|
|quack()//鸭子叫的方法|
|swim()//鸭子游泳的方法|
|display()//鸭子外观的方法|
如果是传统的面向对象思想，所有的鸭子都会叫，也都会游泳，所以quack()方法和swim()方法放在超类Duck中实现，而由于每只鸭子的外观不同，所以display()方法在Duck类中设计成抽象方法，由具体的鸭子继承超类后自己去实现。比如现在有绿头鸭MallardDuck和红头鸭ReaheadDuck，它们继承超类Duck，所以有了quack和swim的功能，同时实现display方法从而有了不同的外观。

但是这样设计有一个问题：
需求开始增加，经过分析，发现鸭子都会飞，所以要给鸭子加上飞的功能。最普通的方法是在Duck超类中加一个fly()的方法并实现。这样每一个鸭子继承Duck之后就有了fly()的功能。但是发现需求有误，实际上并不是所有鸭子都会飞。比如尖叫鸭（类似于尖叫鸡），会叫也会游泳，但是尖叫鸭的叫声是“吱吱”，而不是普通鸭子的“咕咕”。所以尖叫鸭继承Duck之后首先要覆盖掉超类中的quack()方法。最严重的问题是尖叫鸭不会飞，但是继承Duck后它有了飞的功能，这肯定不对。同样的方法就是子类继承超类之后覆盖掉Duck中的fly()方法。但是，如果以后还要加入另一种诱饵鸭子，它其实是一个木头假鸭，不会叫也不会飞。难道又要覆盖掉quack()方法和fly()方法吗？这样太不优雅了。可以看到用继承的方式来提供Duck的行为，会使代码牵一发动全身，不便于维护，也不便于扩展。

所以需要一个更清晰的方法，让“某些”而不是全部鸭子可飞或者可叫。
很自然的一种想法是把飞好叫的行为从Duck超类拿出来，另外设计成接口：Flyable和Quackable。

| Duck//抽象类 | | Flyable//接口|  | Quackable//接口|
|:------| |:--------|  |:--------|
|swim()//普通方法| | fly()//接口中的抽象方法| | quack()//接口中的抽象方法|
|display()//抽象方法||||||
这样尖叫鸭可以继承Duck超类，同时实现Quackable接口，而普通鸭子可以继承Duck超类并同时实现Flyable和Quack接口，但是可以看到，每实现一次接口就要重写一此接口中的方法，重复的代码就会急剧增加。显然这不是一种好的方法。

可以看出造成目前这种困境的原因是需求一直在改变。而现实中需求的改变又恰恰是经常会出现的问题。这个时候就要**找出应用中可能需要经常变化的地方，把他们独立出来，不要和那些不需要变化的代码混在一起。也就是把会变化的部分取出并封装起来，好让其他部分不会受到影响。**
那么具体到这个例子中要怎么做呢？首先在原始的Duck类中把fly()和quack()拿掉，然后建立两组类，一类与fly相关，一类与quack相关，它们分别实现FlyBehavior和QuackBehavior接口。每一组类将实现各自的动作，可能有一个类实现Quackbehhavior接口完成了“咕咕叫”(Quack类)，另一个类实现Quackbehhavior接口完成了“吱吱叫”(SQuack类)，还有一个类实现Quackbehhavior接口完成了“安静”(MuteQuack类)。另外还有一个类实现FlyBehavior接口完成了“飞”的功能（FlyWithSwing类），还有一个类实现FlyBehavior接口完成了“不能飞”（FlyNoWay类）。
但仅仅是这样还不够，因为Java中并没有多继承，如果继承了Duck超类，就不能继承那些实现了FlyBehavior或QuackBehavior的具体实现类。所以要想一种办法把Duck类和我们分离出来的行为整合起来。其实就是在Duck类中添加FlyBehavior和QuackBehavior的属性：

|Duck|
|:-------------------|
|FlyBehavior flyBehavior//飞属性    |
|QuackBehavior quackBehavior//叫属性|
|swim()//鸭子游泳的方法            |
|display()//鸭子外观的方法         |
|performFly()//飞行方法            |
|performQuack()//叫方法            |
```java
public class Duck{
    FlyBehavior flyBehavior;
    QuackBehavior quackBehavior;
    
    //set、get方法
    ...
    public abstract void display();
    public void swim(){
        ...
    }
    public void performFly(){
        flyBehavior.fly();
    }
    public void performQuack(){
        quackBehavior.quack();
    }
}
```
这样当我们实现尖叫鸭的时候：
```java
public class ScreamDuck extends Duck{
    public ScreamDuck(){
        quackBehavior = new SQuack();
        flyBehavior = new FlyNoWay();
    }
    @override
    public void display(){
        ...
    }
}
```
这样就实现了一个尖叫鸭。而普通绿头鸭子就是：
```java
public class MallardDuck extends Duck{
    public MallardDuck(){
        quackBehavior = new Quack();
        flyBehavior = new FlyWithSwing();
    }
    @override
    public void display(){
        ...
    }
}
```
这样维护和扩展都容易多了。

这就是**策略模式**，
**
**策略模式的定义：**
========

**
定义了算法簇，分别封装起来，让它们之间可以互相替换。这个模式可以让算法的变化独立于使用算法的客户。
**策略模式的结构：**
======
![策略模式的结构][1]
  [1]: http://my.csdn.net/uploads/201205/11/1336732187_4598.jpg
**
**策略模式的组成：**
=======
**环境类(Context)**:用一个ConcreteStrategy对象来配置。维护一个对Strategy对象的引用。可定义一个接口来让Strategy访问它的数据。
**抽象策略类(Strategy)**:定义所有支持的算法的公共接口。Context使用这个接口来调用某ConcreteStrategy定义的算法。
**具体策略类(ConcreteStrategy)**:以Strategy接口实现某具体算法。

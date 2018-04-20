---
title: <font color=#0099ff face="微软雅黑">Effective Java 用私有构造器或者枚举类型强化Singleton属性（单例类型）</font>
date: 2017-07-17 09:43:37
categories: java读书笔记
tags: [Java,经验之谈,高效开发,Singleton,单例模式]
---

Singleton指仅仅被实例化一次的类。Singleton通常用来代表那些本质上唯一的系统组件。实现Singleton有多种方法。但是一般的思路都是把构造器保持为私有的，并导出公有的静态成员，以便允许客户端能够访问该类的唯一实例。第一种常见思路：

```java
//Singleton with public final field
public class Elvis {
	public static final Elvis INSTANCE = new Elvis();

	private Elvis() {
	    ...
	}

	public void leaveTheBuilding() {
		...
	}
}
```
在这种思路中，共有静态成员是个final域，私有构造器仅被调用一次，用来实例化公有的静态final域Elvies.INSTANCE。由于缺少共有的或者受保护的构造器，所以保证了Elvies的全局唯一性:一旦Elvis类被实例化，只会存在一个Elvis实例。客户端的任何行为都不会改变这一点，但要注意的是享有特权的客户端可以借助AccessibleObject.setAccessible方法，通过反射机制调用私有构造器。如果需要抵御这种攻击，可以修改构造器，让它在被要求创建第二个实例的时候抛出异常。使用：

```java
// This code would normally appear outside the class!
public static void main(String[] args) {
	Elvis elvis = Elvis.INSTANCE;
	elvis.leaveTheBuilding();
}
```
第二种实现思路中，共有的成员是个静态工厂方法：

```java
public class Elvis {
	private static final Elvis INSTANCE = new Elvis();

	private Elvis() {
	    ...
	}

	public static Elvis getInstance() {
		return INSTANCE;
	}

	public void leaveTheBuilding() {
		...
	}
}
```
对于静态方法Elvis.getInstance的所有调用，都会返回同一个对象引用，所以永远不会创建其他的Elvis实例，但是同样可以通过反射机制来调用私有构造器。

共有域方法的主要好处在于组成类的成员的声明很清楚地表明了这个类是一个Singleton;公有的静态域是final的，所以该域将总是包含相同的对象引用。

为了使 用上述两种方法实现的Singleton类变成是可序列化的，仅仅实现Serializable是不够的。为了维护并保证Singleton，必须声明所有实例域都是瞬时（transient）的，并提供一个readResolve方法。否则，每次反序列化一个序列化的实例时都会创建一个新的实例。为了防止这种情况，要在Elvies类中加入下面这个readResovle方法：

```java
public class Elvis implements Serializable {
	public static final Elvis INSTANCE = new Elvis();

	private Elvis() {
	}

	public void leaveTheBuilding() {
		System.out.println("Whoa baby, I'm outta here!");
	}

	private Object readResolve() {
		// Return the one true Elvis and let the garbage collector
		// take care of the Elvis impersonator.
		return INSTANCE;
	}

	// This code would normally appear outside the class!
	public static void main(String[] args) {
		Elvis elvis = Elvis.INSTANCE;
		elvis.leaveTheBuilding();
	}
}
```
从Java 1.5之后，实现Singeton还有第三种方法。只需编写一个包含单个元素的枚举类型：

```java
public enum Elvis {
	INSTANCE;

	public void leaveTheBuilding() {
		System.out.println("Whoa baby, I'm outta here!");
	}

	// This code would normally appear outside the class!
	public static void main(String[] args) {
		Elvis elvis = Elvis.INSTANCE;
		elvis.leaveTheBuilding();
	}
}
```
这种方法在功能上与公有域方法相近，但是更加简洁，无偿地提供了序列化机制，绝对防止多次实例化，即使是在面对复杂的序列化或者反射攻击的时候。单元素的枚举类型已经成为实现Singleton的最佳方法。






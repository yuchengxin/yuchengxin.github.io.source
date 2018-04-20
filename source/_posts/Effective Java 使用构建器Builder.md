---
title: <font color=#0099ff face="微软雅黑">Effective Java 使用构建器Builder</font>
date: 2017-07-14 08:43:37
categories: java读书笔记
tags: [Java,经验之谈,高效开发, Builder]
---

如果一个类有大量**可选的参数**，如果这个时候使用构造器就需要写大量有不同参数个数的构造器。举个例子，用一个类来表示包装食品外面显示的营养成分标签。这些标签中有一些是必须的，比如每份的含量，每罐的含量以及每份的卡路里。还有超过20多个可选的参数，比如总脂肪量、饱和脂肪量、转化脂肪、胆固醇、钠等等。大多数的产品在某几个可选参数中都会有非零的值。

对于这样的类，传统方法是采用重叠构造器模式，在这种模式下，第一个构造器由所有必要参数来构造，第二个构造器多加一个可选参数，第三个多加两个可选参数，直到最后一个构造器包含所有可选参数。如下所示，为了简便，只显示四个可选域：

```java
/**
* Telescoping constructor pattern - does not scale well
*/

public class NutritionFacts {
    private final int servingSize;         //(ml)                required
    private final int servings;            //(per container)     required
    private final int calories;            //                    optional
    private final int fat;                 //(g)                 optional
    private final int sodium;              //(mg)                optional
    private final int carbohydrate;        //(g)                 optional
    
    public NutritionFacts(int servingSize, int servings) {
        this(servingSize, servings, 0);
    }
    
    public NutritionFacts(int servingSize, int servings, int calories) {
        this(servingSize, servings, calories, 0);
    }
    
    public NutritionFacts(int servingSize, int servings, int calories, int fat) {
        this(servingSize, servings, calories, fat, 0);
    }
    
    public NutritionFacts(int servingSize, int servings, int calories,
            int fat, int sodium) {
        this(servingSize, servings, calories, fat, sodium, 0);
    }
    
    public NutritionFacts(int servingSize, int servings int calories,
            int fat, int sodium, int carbohydrate) {
        this.servingSize = servingSize;
        this.servings = servings;
        this.calories = calories;
        this.fat = fat;
        this.sodium = sodium;
        this.carbohydrate = carbohydrate;
    }
}
```
如果参数非常多，就要写大量的构造器。在使用的时候一不小心颠倒了了其中两个参数的顺序，并不能被及时的发现，但是程序运行的时候则不是正确的结果。这个时候就可以使用Builder模式了。不直接生成想要的对象，而是让客户端利用所有必要的参数调用构造器，得到一个builder对象。然后客户端在builder对象上调用类似于setter的方法，来设置每个相关的可选参数。最后，客户端调用无参的build方法来生成不可变的对象。这个builder是它构建的类的静态成员类。如下：

```java
public class NutritionFacts {
	private final int servingSize;
	private final int servings;
	private final int calories;
	private final int fat;
	private final int sodium;
	private final int carbohydrate;

	public static class Builder {
		// Required parameters
		private final int servingSize;
		private final int servings;

		// Optional parameters - initialized to default values
		private int calories = 0;
		private int fat = 0;
		private int carbohydrate = 0;
		private int sodium = 0;

		public Builder(int servingSize, int servings) {
			this.servingSize = servingSize;
			this.servings = servings;
		}

		public Builder calories(int val) {
			calories = val;
			return this;
		}

		public Builder fat(int val) {
			fat = val;
			return this;
		}

		public Builder carbohydrate(int val) {
			carbohydrate = val;
			return this;
		}

		public Builder sodium(int val) {
			sodium = val;
			return this;
		}

		public NutritionFacts build() {
			return new NutritionFacts(this);
		}
		
		private NutritionFacts(Builder builder) {
    		servingSize = builder.servingSize;
    		servings = builder.servings;
    		calories = builder.calories;
    		fat = builder.fat;
    		sodium = builder.sodium;
    		carbohydrate = builder.carbohydrate;
	    }
	}
```
注意NutritionFacts是不可变的，所有的默认参数都单独放在一个地方。builder的setter方法返回builder本身，以便可以把调用链接起来,如果fat和sodium不需要，那么可以这样创建：

```java
public static void main(String[] args) {
		NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8)
				.calories(100)
				.carbohydrate(27)
				.build();
		...
	}
```
设置了参数的builder生成了一个很好的抽象工厂，换句话说，客户端可以将这样一个builder传给方法，使该方法能够为客户端创建一个或者多个对象。

```java
public interface Builder<T> {
    public T build();
}
```
带有Builder实例的方法通常利用有限制的通配符类型来约束构建器的类型参数。下面就是构建每个节点的方法，它利用一个客户端提供的Builder实例来构建树:

```java
Tree BuilderTree(Builder<？ extends Node> nodeBuilder) {...}
```

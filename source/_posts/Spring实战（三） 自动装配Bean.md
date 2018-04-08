---
title: <font color=#0099ff size=6 face="微软雅黑">Spring实战（三） 自动装配Bean</font>
date: 2017-06-09 08:53:26
categories: java读书笔记
tags: [java,Spring,Bean,自动装配]
---

在Spring中，对象无需自己查找或创建与其所关联的其他对象。相反，容器负责把需要相互协作的对象引用赋予各个对象。例如，一个订单管理组件需要信用卡认证组件，但它不需要自己创建信用卡认证组件。订单管理组件只需要表明自己两手空空，容器就会主动赋予它一个信用卡认证组件。创建应用对象之间协作关系的行为通常称为装配（wiring），这也是依赖注入（DI）的本质。

Spring配置
===========
Spring容器负责创建应用程序中的bean并通过DI来协调这些对象之间的关系。但是，作为开发人员，你需要告诉Spring要创建哪些bean并且如何将其装配在一起。当描述bean如何进行装配时，Spring具有非常大的灵活性，它提供了三种主要的装配机制：

 - 在XML中进行显式配置。
 - 在Java中进行显式配置。
 - 隐式的bean发现机制和自动装配。

Spring的配置风格是可以互相搭配的，所以你可以选择使用XML装配一些bean，使用Spring基于Java的配置（JavaConfig）来装配另一些bean，而将剩余的bean让Spring去自动发现。
即便如此，我的建议是尽可能地使用自动配置的机制。显式配置越少越好。当你必须要显式配置bean的时候（比如，有些源码不是由你来维护的，而当你需要为这些代码配置bean的时候），我推荐使用类型安全并且比XML更加强大的JavaConfig。最后，只有当你想要使用便利的XML命名空间，并且在JavaConfig中没有同样的实现时，才应该使用XML。

自动装配Bean
========
Spring从两个角度来实现自动化装配：

- 组件扫描（component scanning）：Spring会自动发现应用上下文中所创建的bean。
- 自动装配（autowiring）：Spring自动满足bean之间的依赖。

为了阐述组件扫描和装配，我们需要创建几个bean，它们代表了一个音响系统中的组件。首先，要创建CompactDisc类，Spring会发现它并将其创建为一个bean。然后，会创建一个CDPlayer类，让Spring发现它，并将CompactDisc bean注入进来。

**创建可被发现的Bean**
现在有一个CD播放器，需要插入CD才能播放。也就是说CD播放器依赖于CD才能完成它的使命。
先写一个CompactDisc类，它是定义CD的一个接口：
```java
package soundsystem

public interface CompactDisc {
    void play;
}
```
然后我们可以创建多个CD的实现，比如可以是摇滚类的也可以是古典类的，它们都要实现CD的上层接口：CompactDisc。这里只创建一个实现——SgtPeppers类：
```java
package soundsystem;
import org.springframework.stereotype.Component;

@Component
public class SgtPeppers implements CompactDisc {

  private String title = "Sgt. Pepper's Lonely Hearts Club Band";  
  private String artist = "The Beatles";
  
  public void play() {
    System.out.println("Playing " + title + " by " + artist);
  }
  
}
```
SgtPeppers类上使用了@Component注解。这个简单的注解表明该类会作为组件类，并告知Spring要为这个类创建bean。没有必要显式配置SgtPeppers bean，因为这个类使用了@Component注解，所以Spring会为你把事情处理妥当。
不过，组件扫描默认是不启用的。我们还需要显式配置一下Spring，从而命令它去寻找带有@Component注解的类，并为其创建bean,CDPlayerConfig类完成了这个工作：
```java
package soundsystem;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Configuration
@ComponentScan
public class CDPlayerConfig { 
}
```
类CDPlayerConfig通过Java代码定义了Spring的装配规则。但是CDPlayerConfig类并没有显式地声明任何bean，而是使用了@ComponentScan注解，这个注解能够在Spring中启用组件扫描。如果没有其他配置的话，@ComponentScan默认会扫描与配置类相同的包。因为CDPlayerConfig类位于soundsystem包中，因此Spring将会扫描这个包以及这个包下的所有子包，查找带有@Component注解的类。这样的话，就能发现CompactDisc，并且会在Spring中自动为其创建一个bean。

这个工作也可以用xml配置来完成：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:context="http://www.springframework.org/schema/context"
  xmlns:c="http://www.springframework.org/schema/c"
  xmlns:p="http://www.springframework.org/schema/p"
  xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

  <context:component-scan base-package="soundsystem" />

</beans>
```
现在就可以创建一个测试类来试着尝试现有的功能了。在这个JUnit测试类中，先会创建Spring上下文，然后判断CompactDisc是不是真的创建出来了：
```java
package soundsystem;

import static org.junit.Assert.*;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes=CDPlayerConfig.class)
public class CDPlayerTest {
  
  @Autowired
  private CompactDisc cd;
  
  @Test
  public void cdShouldNotBeNull() {
    assertNotNull(cd);
  }

}

```
CDPlayerTest使用了Spring的SpringJUnit4ClassRunner，以便在测试开始的时候自动创建Spring的应用上下文。注解@ContextConfiguration会告诉它需要在CDPlayerConfig中加载配置。因为CDPlayerConfig类中包含了@ComponentScan，因此最终的应用上下文中应该包含CompactDisc bean。为了证明这一点，在测试代码中有一个CompactDisc类型的属性，并且这个属性带有@Autowired注解，以便于将CompactDiscbean注入到测试代码之中（稍后，我会讨论@Autowired）。最后，会有一个简单的测试方法断言cd属性不为null。如果它不为null的话，就意味着Spring能够发现CompactDisc类，自动在Spring上下文中将其创建为bean并将其注入到测试代码之中。

尽管我们只用它创建了一个bean，但同样是这么少的配置能够用来发现和创建任意数量的bean。在soundsystem
包及其子包中，所有带有@Component注解的类都会创建为bean。只添加一行@Component Scan注解就能自动创建无数个bean，这种权衡还是很划算的。

但是使用@ComponentScan和@Component组件扫描还能做别的事情：
**为组件扫描的Bean命名**
Spring应用上下文中所有的bean都会给定一个ID。在前面的例子中，尽管我们没有明确地为SgtPeppersbean设置ID，但Spring会根据类名为其指定一个ID。具体来讲，这个bean所给定的ID为sgtPeppers，也就是将类名的第一个字母变为小写。如果想为这个bean设置不同的ID，你所要做的就是将期望的ID作为值传递给@Component
注解。比如说，如果想将这个bean标识为lonelyHeartsClub，那么你需要将SgtPeppers类的@Component注解配置为如下所示：
```java
@Component("lonelyHeartsClub")
public class SgtPeppers implements CompactDisc {
    ...
}
```
还有另外一种为bean命名的方式，这种方式不使用@Component注解，而是使用Java依赖注入规范（Java Dependency Injection）中所提供的@Named注解来为bean设置ID：
```java
package soundsystem;

import javax.inject.Named;

@Named("lonelyHeartsClub")
public class SgtPeppers implements CompactDisc {
    ...
}
```
Spring支持将@Named作为@Component注解的替代方案。两者之间有一些细微的差异，但是在大多数场景中，它们是可以互相替换的。

**设置组件扫描的的基础包**
到现在为止，我们没有为@ComponentScan设置任何属性。这意味着，按照默认规则，它会以配置类所在的包作为基础包（basepackage）来扫描组件。但是，如果你想扫描不同的包，那该怎么办呢？或者，如果你想扫描多个基础包，那又该怎么办呢？
有一个原因会促使我们明确地设置基础包，那就是我们想要将配置类放在单独的包中，使其与其他的应用代码区分开来。如果是这样的话，那默认的基础包就不能满足要求了。要满足这样的需求其实也完全没有问题！为了指定不同的基础包，你所需要做的就是在@ComponentScan的value属性中指明包的名称：
```java
@Configuration
@ComponentScan("soundsystem")
public class CDPlayerConfig {}
```
如果你想更加清晰地表明你所设置的是基础包，那么你可以通过basePackages属性进行配置：
```java
@Configuration
@ComponentScan(basePackages="soundsystem")
public class CDPlayerConfig {}
```
可能你已经注意到了basePackages属性使用的是复数形式。如果你揣测这是不是意味着可以设置多个基础包，那么恭喜你猜对了。如果想要这么做的话，只需要将basePackages属性设置为要扫描包的一个数组即可：
```java
@Configuration
@ComponentScan(basePackages={"soundsystem","video"})
public class CDPlayerConfig {}
```
在上面的例子中，所设置的基础包是以String类型表示的。我认为这是可以的，但这种方法是类型不安全（not typesafe）的。如果你重构代码的话，那么所指定的基础包可能就会出现错误了。除了将包设置为简单的String类型之外，@ComponentScan还提供了另外一种方法，那就是将其指定为包中所包含的类或接口：
```java
@Configuration
@ComponentScan(basePackageClasses={CDPlayer.class, DVDPlayer.class})
public class CDPlayerConfig {}
```
可以看到，basePackages属性被替换成了basePackageClasses。同时，我们不是再使用String类型的名称来指定包，为basePackageClasses属性所设置的数组中包含了类。这些类所在的包将会作为组件扫描的基础包。


在你的应用程序中，如果所有的对象都是独立的，彼此之间没有任何依赖，就像SgtPeppersbean这样，那么你所需要的可能就是组件扫描而已。但是，很多对象会依赖其他的对象才能完成任务。这样的话，我们就需要有一种方法能够将组件扫描得到的bean和它们的依赖装配在一起。要完成这项任务，我们需要了解一下Spring自动化配置的另外一方面内容，那就是自动装配。

**通过为bean添加注解实现自动装配**
------
简单来说，自动装配就是让Spring自动满足bean依赖的一种方法，在满足依赖的过程中，会在Spring应用上下文中寻找匹配某个bean需求的其他bean。为了声明要进行自动装配，我们可以借助Spring的@Autowired注解。

下面完成CDPlayer类：
```java
package soundsystem;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

@Component
public class CDPlayer implements MediaPlayer {
  private CompactDisc cd;

  @Autowired
  public CDPlayer(CompactDisc cd) {
    this.cd = cd;
  }

  public void play() {
    cd.play();
  }

}
```
它的构造器上添加了@Autowired注解，这表明当Spring创建CDPlayerbean的时候，会通过这个构造器来进行实例化并且会传入一个可设置给CompactDisc类型的bean。

@Autowired注解不仅能够用在构造器上，还能用在属性的Setter方法上。比如说，如果CDPlayer有一个setCompactDisc()方法，那么可以采用如下的注解形式进行自动装配：
```java
@Autowired
public void setCompactDisc(CompactDisc cd) {
    this.cd = cd;
}
```

@Autowired注解可以用在类的任何方法上。假设CDPlayer类有一个insertDisc()方法，那么@Autowired能够像在setCompactDisc()上那样，发挥完全相同的作用：
```java
@Autowired
public void insertDisc(CompactDisc cd) {
    this.cd = cd;
}
```
不管是构造器、Setter方法还是其他的方法，Spring都会尝试满足方法参数上所声明的依赖。假如有且只有一个bean匹配依赖需求的话，那么这个bean将会被装配进来。如果没有匹配的bean，那么在应用上下文创建的时候，Spring会抛出一个异常。为了避免异常的出现，你可以将@Autowired的required属性设置为false：
```java
@Autowired(required=false)
public CDPlayer(CompactDisc cd) {
    this.cd = cd;
}
```
将required属性设置为false时，Spring会尝试执行自动装配，但是如果没有匹配的bean的话，Spring将会让这个bean处于未装配的状态。但是，把required属性设置为false时，你需要谨慎对待。如果在你的代码中没有进行null检查的话，这个处于未装配状态的属性有可能会出现NullPointerException。如果有多个bean都能满足依赖关系的话，Spring将会抛出一个异常，表明没有明确指定要选择哪个bean进行自动装配。

@Autowired是Spring特有的注解。如果你不愿意在代码中到处使用Spring的特定注解来完成自动装配任务的话，那么你可以考虑将其替换为@Inject：
```java
package soundsystem;
import javax.inject.Inject;
importjavax.inject.Named;

@Named
public cladd CDPlayer{
    ...
    
    @Inject
    public CDPlayer(CompactDisc cd){
        this.cd = cd;
    }
    
    ...
}
```
@Inject注解来源于Java依赖注入规范，该规范同时还为我们定义了@Named注解。在自动装配中，Spring同时支持@Inject和@Autowired。尽管@Inject和@Autowired之间有着一些细微的差别，但是在大多数场景下，它们都是可以互相替换的。

现在，我们已经在CDPlayer的构造器中添加了@Autowired注解，Spring将把一个可分配给CompactDisc类型的bean自动注入进来。为了验证这一点，让我们修改一下CDPlayerTest，使其能够借助CDPlayerbean播放CD：
```java
package soundsystem;

import static org.junit.Assert.*;

import org.junit.Rule;
import org.junit.Test;
import org.junit.contrib.java.lang.system.StandardOutputStreamLog;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes=CDPlayerConfig.class)
public class CDPlayerTest {

  @Rule
  public final StandardOutputStreamLog log = new StandardOutputStreamLog();

  @Autowired
  private MediaPlayer player;
  
  @Autowired
  private CompactDisc cd;
  
  @Test
  public void cdShouldNotBeNull() {
    assertNotNull(cd);
  }

  @Test
  public void play() {
    player.play();
    assertEquals(
        "Playing Sgt. Pepper's Lonely Hearts Club Band by The Beatles\n", 
        log.getLog());
  }

}
```
现在，除了注入CompactDisc，我们还将CDPlayerbean注入到测试代码的player成员变量之中（它是更为通用的
MediaPlayer类型）。在play()测试方法中，我们可以调用CDPlayer的play()方法，并断言它的行为与你的预期一致。


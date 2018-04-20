---
title: <font color=#0099ff face="微软雅黑">Spring实战（一） 依赖注入和面向切面编程</font>
date: 2017-06-02 09:15:26
categories: java读书笔记
tags: [java,Spring,依赖注入,面向切面编程,IoC,AOP]
---

Spring用bean或者JavaBean来表示应用组件，但并不意味着Spring组件必须要遵循JavaBean规范。一个Spring组件可以是任何形式的POJO。以下都采用JavaBean的广泛定义，即POJO的同义词。

Spring的根本使命就是简化Java开发，而为了降低Java开发的复杂性，Spring采取了以下4种关键策略：
- 基于POJO的轻量级和最小侵入性编程；
- 通过依赖注入和面向接口实现松耦合；
- 基于切面和惯例进行声明式编程；
- 通过切面和模板减少样板式代码。

依赖注入
======
**通过DI，对象的依赖关系将由系统中负责协调各对象的第三方组件在创建对象的时候进行设定。对象无需自行创建或管理它们的依赖关系。依赖注入主要是实现松耦合。**

假如现在有一个骑士DamselRescuingKnight：
```java
public class DamselRescuingKnight implements Knight {

  private RescueDamselQuest quest;

  public DamselRescuingKnight() {
    this.quest = new RescueDamselQuest();
  }

  public void embarkOnQuest() {
    quest.embark();
  }
  
}
```
可以看到DamselRescuingKnight这个类中有一个成员变量RescueDamselQuest，这是一个具体的探险任务类型，所以DamselRescuingKnight这个类型的骑士就只能执行RescueDamselQuest探险任务。这使得DamselRescuingKnight紧密地和RescueDamselQuest耦合到了一起，因此极大地限制了这个骑士执行探险的能力。
但是如果我们这样去实现：
```java
public class BraveKnight implements Knight {

  private Quest quest;

  public BraveKnight(Quest quest) {
    this.quest = quest;
  }

  public void embarkOnQuest() {
    quest.embark();
  }

}
```
可以看到BraveKnight没有自行创建具体的探险任务，而是在构造的时候把探险任务作为构造器参数传入。这是依赖注入的方式之一，即构造器注入（constructor injection）。更重要的是，传入的探险类型是Quest，也就是所有探险任务都必须实现的一个接口。所以，BraveKnight能够响应RescueDamselQuest、 SlayDragonQuest、 MakeRoundTableRounderQuest等任意的Quest实现。这就是DI所带来的最大收益——松耦合。

现在BraveKnight类可以接受你传递给它的任意一种Quest的实现，但该怎样把特定的Query实现传给它呢？假设，希望BraveKnight所要进行探险任务是杀死一只怪龙，那么SlayDragonQuest也许挺合适的。但是要怎么把SlayDragonQuest传给BraveKnight呢？

**创建应用组件之间协作的行为通常称为装配（wiring）。Spring有多种装配bean的方式，采用XML是很常见的一种装配方式。**这里就可以用knights.xml把SlayDragonQuest传给BraveKnight：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans 
      http://www.springframework.org/schema/beans/spring-beans.xsd">

  <bean id="knight" class="sia.knights.BraveKnight">
    <!--注入Quest Bean-->
    <constructor-arg ref="quest" /> 
  </bean>

   <!--创建SlayDragonQuest-->
  <bean id="quest" class="sia.knights.SlayDragonQuest">
    <constructor-arg value="#{T(System).out}" />
  </bean>

</beans>
```
在这里，BraveKnight和SlayDragonQuest被声明为Spring中的bean。就BraveKnight bean来讲，它在构造时传入了对SlayDragonQuestbean的引用，将其作为构造器参数。同时，SlayDragonQuestbean的声明使用了Spring表达式语言（SpringExpressionLanguage），将System.out（这是一个PrintStream）传入到了SlayDragonQuest的构造器中。

**Spring还支持使用Java来描述配置。**下面的做法和上面xml的功能是一样的，都是将SlayDragonQuest注入到BraveKnight：
```java
package sia.knights.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import sia.knights.BraveKnight;
import sia.knights.Knight;
import sia.knights.Quest;
import sia.knights.SlayDragonQuest;

@Configuration
public class KnightConfig {

  @Bean
  public Knight knight() {
    return new BraveKnight(quest());
  }
  
  @Bean
  public Quest quest() {
    return new SlayDragonQuest(System.out);
  }

}
```
不管你使用的是基于XML的配置还是基于Java的配置，DI所带来的收益都是相同的。尽管BraveKnight依赖于Quest，但是它并不知道传递给它的是什么类型的Quest，也不知道这个Quest来自哪里。只有Spring通过它的配置，能够了解这些组成部分是如何装配起来的。这样的话，就可以在不改变所依赖的类的情况下，修改依赖关系。

现在已经声明了BraveKnight和Quest的关系，接下来我们只需要装载XML配置文件，并把应用启动起来。Spring通过应用上下文（Application Context）装载bean的定义并把它们组装起来。Spring应用上下文全权负责对象的创建和组装。Spring自带了多种应用上下文的实现，它们之间主要的区别仅仅在于如何加载配置。

因为knights.xml中的bean是使用XML文件进行配置的，所以选择Class Path Xml Application Context作为应用上下文相对是比较合适的。该类加载位于应用程序类路径下的一个或多个XML配置文件。下面程序中的main()方法调用ClassPathXmlApplicationContext加载knights.xml，并获得Knight对象的引用。
```java
package sia.knights;

import org.springframework.context.support.ClassPathXmlApplicationContext;

public class KnightMain {

  public static void main(String[] args) throws Exception {
    //加载Spring上下文
    ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("META-INF/spring/knight.xml");
    //获取Knight Bean
    Knight knight = context.getBean(Knight.class);
    //使用Knight
    knight.embarkOnQuest();
    context.close();
  }

}
```
这里的main()方法基于knights.xml文件创建了Spring应用上下文。随后它调用该应用上下文获取一个ID为knight的bean。得到Knight对象的引用后，只需简单调用embarkOnQuest()方法就可以执行所赋予的探险任务了。注意这个类完全不知道我们的英雄骑士接受哪种探险任务，而且完全没有意识到这是由BraveKnight来执行的。只有knights.xml文件知道哪个骑士执行哪种探险任务。

面向切面编程
======
**DI能够让相互协作的软件组件保持松散耦合，而面向切面编程(aspect-oriented programming，AOP)允许你把遍布应用各处的功能分离出来形成可重用的组件。**

系统由许多不同的组件组成，每一个组件各负责一块特定功能。除了实现自身核心的功能之外，这些组件还经常承担着额外的职责。诸如日志、事务管理和安全这样的系统服务经常融入到本身具有核心业务逻辑的组件中去，这些系统服务通常被称为横切关注点，因为它们会跨越系统的多个组件。如果将这些关注点分散到多个组件中去，你的代码将会带来双重的复杂性：

- 实现系统关注点功能的代码将会重复出现在多个组件中。这意味着如果你要改变这些关注点的逻辑，必须修改各个模块中的相关实现。即使你把这些关注点抽象为一个独立的模块，其他模块只是调用它的方法，但方法的调用还是会重复出现在各个模块中。
- 组件会因为那些与自身核心业务无关的代码而变得混乱。一个向地址簿增加地址条目的方法应该只关注如何添加地址，而不应该关注它是不是安全的或者是否需要支持事务。

AOP能够使这些服务模块化，并以声明的方式将它们应用到它们需要影响的组件中去。所造成的结果就是这些组件会具有更高的内聚性并且会更加关注自身的业务，完全不需要了解涉及系统服务所带来复杂性。总之，AOP能够确保POJO的简单性。

**AOP的使用：**
每一个人都熟知骑士所做的任何事情，这是因为吟游诗人用诗歌记载了骑士的事迹并将其进行传唱。假设我们需要使用吟游诗人这个服务
类来记载骑士的所有事迹。Minstrel类：
```java
package sia.knights;

import java.io.PrintStream;

public class Minstrel {

      private PrintStream stream;
      
      public Minstrel(PrintStream stream) {
        this.stream = stream;
      }

      //探险之前调用
      public void singBeforeQuest() {
        stream.println("Fa la la, the knight is so brave!");
      }
    
      //探险之后调用
      public void singAfterQuest() {
        stream.println("Tee hee hee, the brave knight " +
        		"did embark on a quest!");
      }

}
```
Minstrel是只有两个方法的简单类。在骑士执行每一个探险任务之前，singBeforeQuest()方法会被调用；在骑士完成探险任务之后，singAfterQuest()方法会被调用。在这两种情况下，Minstrel都会通过一个PrintStream类来歌颂骑士的事迹，这个类是通过构造器注入进来的。我们适当做一下调整从而让BraveKnight可以使用Minstrel：
```java
package sia.knights;
  
public class BraveKnight implements Knight {

  private Quest quest;
  private Minstrel minstrel;

  public BraveKnight(Quest quest) {
    this.quest = quest;
    this.minstrel = minstrel;
  }

  public void embarkOnQuest() {
    minstrel.singBeforeQuest();
    quest.embark();
    minstrel.singAfterQuest();
  }

}
```
这应该可以达到预期效果。现在，你所需要做的就是回到Spring配置中，声明Minstrel bean并将其注入到
BraveKnight的构造器之中。但是，管理吟游诗人的行为真的是骑士职责范围内的工作吗？在我看来，吟游诗人应该做他份内的事，根本不需要骑士命令他这么做。此外，因为骑士需要知道吟游诗人，所以就必须把吟游诗人注入到BarveKnight类中。这不仅使BraveKnight的代码复杂化了，而且还让我疑惑是否还需要一个不需要吟游诗人的骑士呢？如果Minstrel为null会发生什么呢？我是否应该引入一个空值校验逻辑来覆盖该场景？

但利用AOP，你可以声明吟游诗人必须歌颂骑士的探险事迹，而骑士本身并不用直接访问Minstrel的方法。要将
Minstrel抽象为一个切面，你所需要做的事情就是在一个Spring配置文件中声明它。下面是更新后的knights.xml文件，Minstrel被声明为一个切面：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:aop="http://www.springframework.org/schema/aop"
  xsi:schemaLocation="http://www.springframework.org/schema/aop 
      http://www.springframework.org/schema/aop/spring-aop.xsd
		http://www.springframework.org/schema/beans 
      http://www.springframework.org/schema/beans/spring-beans.xsd">

  <bean id="knight" class="sia.knights.BraveKnight">
    <constructor-arg ref="quest" />
  </bean>

  <bean id="quest" class="sia.knights.SlayDragonQuest">
    <constructor-arg value="#{T(System).out}" />
  </bean>

  <!--声明minstrel bean-->
  <bean id="minstrel" class="sia.knights.Minstrel">
    <constructor-arg value="#{T(System).out}" />
  </bean>

  <!--定义切面-->
  <aop:config>
    <aop:aspect ref="minstrel">
      <aop:pointcut id="embark"
          expression="execution(* *.embarkOnQuest(..))"/>
      <!--声明前置通知-->
      <aop:before pointcut-ref="embark" 
          method="singBeforeQuest"/>
      <!--声明后置通知-->
      <aop:after pointcut-ref="embark" 
          method="singAfterQuest"/>
    </aop:aspect>
  </aop:config>
  
</beans>
```
这里使用了Spring的aop配置命名空间把Minstrel bean声明为一个切面。首先，需要把Minstrel声明为一个bean，然后在< aop:aspect>元素中引用该bean。为了进一步定义切面，声明（使用< aop:before>
）在embarkOnQuest()方法执行前调用Minstrel的singBeforeQuest()方法。这种方式被称为前置通知（before advice）。同时声明（使用< aop:after>）在embarkOnQuest()方法执行后调用singAfterQuest()方法。这种方式被称为后置通知（after advice）。
在这两种方式中，pointcut-ref属性都引用了名字为embank的切入点。该切入点是在前边的< pointcut>
元素中定义的，并配置expression属性来选择所应用的通知。表达式的语法采用的是AspectJ的切点表达式语言。这样Spring在骑士执行探险任务前后就会调用Minstrel的singBeforeQuest()和singAfterQuest()方法。

Minstrel仍然是一个POJO，没有任何代码表明它要被作为一个切面使用。当我们按照上面那样进行配置后，在S
pring的上下文中，Minstrel实际上已经变成一个切面了。其次，也是最重要的，Minstrel可以被应用到BraveKnight中，而BraveKnight不需要显式地调用它。实际上，BraveKnight完全不知道Minstrel的存在。必须还要指出的是，尽管我们使用Spring魔法把Minstrel转变为一个切面，但首先要把它声明为一个Spring bean。能够为其他Spring bean做到的事情都可以同样应用到Spring切面中，例如为它们注入依赖。


---
title: <font color=#0099ff face="微软雅黑">Effective Java 用静态工厂代替构造器</font>
date: 2017-07-12 08:43:37
categories: java读书笔记
tags: [Java,经验之谈,高效开发]
---

对于一个类，要获得它的一个实例，一般而言是提供一个共有的构造器。但是可以考虑用一个静态工厂来代替构造器。要注意的是，这里的静态工厂与设计模式中的工厂方法不同。

来看一看服务提供者框架：
服务提供者框架是指这样一个系统：多个服务提供者实现了某个服务，系统为用户提供多个实现，并把它们从多个实现中解耦出来。

服务提供者框架中有三个重要的组件：服务接口(Service Interface), 这是由提供者定义和实现的。提供者注册API(Provider Registration API)，这是系统用来注册提供者的，因为具体的服务由提供者实现，所以实际上只有注册提供者之后才会有具体的服务。服务访问API(Service Access API)，是用户用来获取服务实例的。还有一个可选的组件：服务提供者接口(Service Provider Interface)
这些提供者负责创建其服务实现的实例。如果没有服务提供者接口，实现就按照类名称注册，并通过反射方式进行实例化。

举一个实例：
服务接口(Service Interface)：

```java
/**
* Service Interface
*/
public interface Service {
	// Service-specific methods go here
}
```
服务提供者接口(Service Provider Interface):

```java
/**
* Service Provider Interface
*/
public interface Provider {
	Service newService();
}
```

```java
/**
* Noninstantiable class for service registration and access
*/
public class Services {
	private Services() {
	} // Prevents instantiation (Item 4)

	// Maps service names to services
	private static final Map<String, Provider> providers = new ConcurrentHashMap<String, Provider>();
	public static final String DEFAULT_PROVIDER_NAME = "<def>";

	// Provider registration API
	public static void registerDefaultProvider(Provider p) {
		registerProvider(DEFAULT_PROVIDER_NAME, p);
	}

	public static void registerProvider(String name, Provider p) {
		providers.put(name, p);
	}

	// Service access API
	public static Service newInstance() {
		return newInstance(DEFAULT_PROVIDER_NAME);
	}

	public static Service newInstance(String name) {
		Provider p = providers.get(name);
		if (p == null)
			throw new IllegalArgumentException(
					"No provider registered with name: " + name);
		return p.newService();
	}
}

```
这个类中包含提供者注册API和服务访问API。

最后是一个测试用例：

```java
public class Test {
    //定义提供者，每一个提供者实现了一钟服务
	private static Provider DEFAULT_PROVIDER = new Provider() {
		public Service newService() {
			return new Service() {
				@Override
				public String toString() {
					return "Default service";
				}
			};
		}
	};

	private static Provider COMP_PROVIDER = new Provider() {
		public Service newService() {
			return new Service() {
				@Override
				public String toString() {
					return "Complementary service";
				}
			};
		}
	};

	private static Provider ARMED_PROVIDER = new Provider() {
		public Service newService() {
			return new Service() {
				@Override
				public String toString() {
					return "Armed service";
				}
			};
		}
	};
	
	public static void main(String[] args) {
		// Providers would execute these lines
		//调用提供者注册API来注册提供者，注册完成之后，对应的提供者就会创建对应的服务
		Services.registerDefaultProvider(DEFAULT_PROVIDER);
		Services.registerProvider("comp", COMP_PROVIDER);
		Services.registerProvider("armed", ARMED_PROVIDER);

		// Clients would execute these lines
		//调用服务访问API来获得服务的实例
		Service s1 = Services.newInstance();
		Service s2 = Services.newInstance("comp");
		Service s3 = Services.newInstance("armed");
		System.out.printf("%s, %s, %s%n", s1, s2, s3);
	}
}
```

理解：在测试用例中先定义一些提供者，之前说过服务是由提供者自己实现的，所以在定义这些提供者的时候要实现具体的服务。
在main函数中去真正使用服务的时候，首先要调用提供者注册API去对定义的提供者进行注册。注册完成之后才会有具体的服务。这个时候就可以通过服务访问API拿到某个提供者提供的具体服务了。

上面示例的运行结果：

```java
Default service, Complementary service, Armed service

Process finished with exit code 0
```






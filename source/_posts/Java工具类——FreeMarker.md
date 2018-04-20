---
title: <font color=#0099ff face="微软雅黑">Java工具——FreeMaker模板引擎</font>
date: 2017-05-26 08:47:26
categories: java工具
tags: [java,FreeMaker,模板引擎]
---
FreeMaker是一个模板引擎，它基于模板和数据来生成输出文本(HTML网页，电子邮件，配置文件，源代码等)。也就是说它可以生成文件，而这些文件的格式是我们指定好的格式。文件中的内容是通过插值来填充的。也就是所谓的模板+数据=输出。

首先来看看FreeMarker模板文件，它是以.ftl为后缀的文件，主要由如下4个部分组成:
1.文本:直接输出的部分
2.注释:<#-- ... -->格式部分,不会输出
3.插值:即${...}或#{...}格式的部分,将使用数据模型中的部分替代输出
4.FTL指令:FreeMarker指定,和HTML标记类似,名字前加#予以区分,不会输出
 
具体编程的话，在Java中，为了使用FreeMarker将数据模型中的值合并到从模板文件生成的模板中,可按如下步骤进行:
1.**创建配置实例**：首先，创建一个Configuration的实例，然后调整它的设置。Configuration实例是存储FreeMarker应用级设置的核心部分。同时，它也处理那些创建和缓存预解析模板的工作。
```java
Configuration cfg = new Configuration(Configuration.VERSION_2_3_22); 
// 指定模板文件从何处加载的数据源，这里设置成一个文件目录。 
cfg.setDirectoryForTemplateLoading(new File("/where/you/store/templates")); 
// 指定模板如何检索数据模型，后面会讲到，先可以这么来用： 
cfg.setObjectWrapper(new DefaultObjectWrapper()); 
//指定异常处理句柄
cfg.setTemplateExceptionHandler(TemplateExceptionHandler.RETHROW_HANDLER);
//指定默认的编码格式
cfg.setDefaultEncoding("UTF-8");
```
这些工作只在应用(可能是servlet)生命周期的开始执行一次。
2.**创建数据模型**：
比如这样一个数据模型:
(root) 
  | 
  +- user = "Big Joe" 
  | 
  +- latest Product 
  　　| 
  　　+- url = "products/greenmouse.html" 
  　　| 
  　　+- name = "green mouse"
用java来构建：
```java
// 创建根哈希表 
Map root = new HashMap(); 
// 在根中放入字符串"user" 
root.put("user", "Big Joe"); 
// 为"latestProduct"创建哈希表 
Map latest = new HashMap(); 
// 将它添加到根哈希表中 
root.put("latestProduct", latest); 
// 在 latest 中放置"url"和"name"  
latest.put("url", "products/greenmouse.html"); 
latest.put("name", "green mouse");
```
现在root就是要填充到模板template中的数据了。
3.**获得模板**
现在有一个模板文件：test.ftl，存放在之前设置的目录中。test.ftl文件中的内容(即模板)如下：
```html
<html> 
<head> 
  <title>Welcome!</title> 
</head> 
<body> 
  <h1>Welcome ${user}!</h1> 
  <p>Our latestproduct:</p> 
  <a href="${latestProduct.url}">${latestProduct.name}</a>! 
</body> 
</html> 
```
在程序中我们可以用freemarker.template.Template来临时存储模板的格式，当我们需要一个模板实例的时候，就可以使用Configuration的getTemplate方法来获取。在之前设置的目录中，我们用test.ftl存储了示例模板，那么就可以这样来做：
```java
Template temp = cfg.getTemplate("test.ftl");
```
当调用这个方法的时候，将会创建一个Template实例,它是test.ftl文件经过读取和解析（编译）之后创建的模板实例。Template实例以解析后的形式存储模板，而不是以源文件的文本形式。 
4.**合并模板和数据模型**
我们都已经知道：数据模型+模板=输出，现在我们已经有了一个数据模型root和一个模板temp了，所以为了得到输出就需要合并它们。这是由模板的process方法完成的。它用数据模型root和Writer对象作为参数，然后向
Writer对象写入产生的内容。为简单起见，这里我们只做标准的输出：
```java
Writer out = new OutputStreamWriter(System.out); 
temp.process(root, out); 
out.flush();
```
上面的代码会向你的终端以数据模板的格式输出数据模型的数据。
完整的代码：
```java
import freemarker.template.*; 
import java.util.*; 
import java.io.*; 
public classTest { 
    public static void main(String[] args) throws Exception { 
        /* 在整个应用的生命周期中，这个工作你应该只做一次。 */  
        /* 创建和调整配置。 */ 
        Configuration cfg = new Configuration(); 
        cfg.setDirectoryForTemplateLoading(new File("/where/you/store/templates")); 
        cfg.setObjectWrapper(new DefaultObjectWrapper()); 
       
        /* 在整个应用的生命周期中，这个工作你可以执行多次 */  
        /* 获取或创建模板*/ 
        Template temp = cfg.getTemplate("test.ftl"); 
        
        /* 创建数据模型 */ 
        Map root = new HashMap(); 
        root.put("user", "Big Joe"); 
        Map latest = new HashMap(); 
        root.put("latestProduct", latest); 
        latest.put("url", "products/greenmouse.html"); 
        latest.put("name", "green mouse"); 
       
        /* 将模板和数据模型合并 */ 
        Writer out = new OutputStreamWriter(System.out);
        temp.process(root, out); 
        out.flush(); 
    } 
} 
```

上面已经讲了基本用法，现在进行更深入的理解：
**Configuration**
-------------

Configuration存储了常用(全局，应用程序级)的设置，定义了想要在所有模板中可用的变量(称为共享变量)。 而且，它会处理 Template 实例的新建和缓存。通常使用一个共享的单实例Configuration对象。

**设置共享变量**，是为所有模板所定义的变量。
```java
cfg.setSharedVariable(String name,Object obj);  
```
比如：
```java
Configuration cfg = new Configuration();
...
cfg.setSahredVariable("wrap", new WrapDirective());
// 使用 ObjectWrapper.DEFAULT_WRAPPER 
cfg.setSharedVariable("company", "Foo Inc.");
```
上面这段代码表示：
在所有使用这个配置的模板中，名为wrap的用户自定义指令。
一个名为company的字符串将会在数据模型的根上可见，这样就不用在根哈希表上一次又一次的添加它们。在传递给Template.process根对象里的变量将会隐藏同名的共享变量。
警告：如果配置对象在多线程环境中使用，不要使用TemplateModel实现类来作为共享变量，因为它是线程不安全的。这也是基于Servlet的Web站点的典型情况。

**配置设置**
配置设置有很多， 例如：locale，number_format，default_encoding， template_exception_handler。
比如说设置编码格式:
```java
cfg.setDefaultEncoding("UTF-8");  
```
那么使用该配置的所有模板的编码格式都是UTF-8。
配置信息可以被想象成3层(Configuration， Template，Environment)，最高层包含特定的值，它为设置信息提供最有效的值。 比如(设置信息A到F仅仅是为这个示例而构想的)：

|| Setting A |Setting B|Setting C|Setting D|Setting E|Setting F|
|:---------:|:-----------:|:-----------:|:-----------:|:-----------:|:-----------:|
|Layer 3: Environment| 1 |-|-|1|-|-|
|Layer 2: Template| 2 |2|-|-|2|-|
|Layer 1: Configuration| 3 |3|3|3|-|-|
配置信息的有效值为：A=1，B=2，C=3，D=1，E=2。 而F的设置则是null，或者在你获取它的时候将抛出异常。

 - Configuration 层： 原则上设置配置信息时使用 Configuration 对象的setter方法，例如：
	```java
	Configuration myCfg = new Configuration(Configuration.VERSION_2_3_23);
	myCfg.setTemplateExceptionHandler(TemplateExceptionHandler.RETHROW_HANDLER);
	myCfg.setDefaultEncoding("UTF-8");
	```
	在真正使用 Configuration 对象 (通常在初始化应用程序时)之前来配置它，后面必须将其视为只读的对象。

 - Template层：对于被请求的本地化信息，也就是模板的locale 设置，它由Configuration.getTemplate(...) 来设置。但是如果我们想控制Template对象来代替freemarker.cache.TemplateCache的话，就应该在Template对象第一次被使用前就设置配置信息，然后就将Template对象视为是只读的。
 - Environment 层：这里有两种配置方法： 
	一是使用Java API：使用 Environment对象的setter方法。当然这需要在模板执行之前来做。但是我们如果直接调用myTemplate.process(...) 就会遇到问题，因为在内部创建Environment对象后立即就执行模板了，导致没有机会来进行设置。这个问题的解决可以用下面两个步骤进行：
	```java
	Environment env = myTemplate.createProcessingEnvironment(root, out);
	env.setLocale(java.util.Locale.ITALY); 
	env.setNumberFormat("0.####");
	env.process();  // process the template
	```
	二是在模板中直接使用setting指令，但是这种方法并不推荐，例如：
	```java
	<#setting locale="it_IT">
	<#setting number_format="0.####">
	```
	在Environment层中，什么时候改变配置信息是没有限制的。

**模板加载**
模板加载器加载基于抽象模板路径下的比如 "index.ftl" 或 "products/catalog.ftl"的原生文本数据对象。加载完毕之后，它会解析文本来生成模板。

 - 内建模板加载器：
	在 Configuration 中可以使用下面的方法来建立三种不同的模板加载器。 (每种方法都会在其内部新建一个模板加载器对象，然后创建 Configuration 实例来使用它。)
	```java
	void setDirectoryForTemplateLoading(File dir);
	```
	```java
	void setClassForTemplateLoading(Class cl, String prefix);
	```
	```java
	void setServletContextForTemplateLoading(Object servletContext, String path);
	```
	上述的第一种方法在磁盘的文件系统上设置了一个明确的目录， 它确定了从哪里加载模板。
	第二种调用方法的参数是一个类和一个路径前缀，参数prefix是给模板的名称来加前缀的，其实就是文件相对于给定类的路径。这个方法顾名思义就是通过获得给定类的相对位置以及之后的prefix参数来获得template文件。这样方便我们将template文件一起打成jar包使用。经验之谈，这里prefix必须以“/”开头。
	第三种调用方式需要Web应用的上下文和一个基路径作为参数，这个基路径是Web应用根路径(WEB-INF目录的上级目录)的相对路径。加载器会从Web应用目录开始加载模板。而且加载方法对没有打包的.war文件也起作用，因为它使用了ServletContext.getResource()方法来访问模板。比如： setServletContextForTemplateLoading(context, "/ftl") 就是 /WebRoot/ftl目录。

 - 从多个位置加载模板:
	如果需要从多个位置加载模板，那就不得不为每个位置都实例化模板加载器对象，将他们封装成一个MultiTemplateLoader的特殊模板加载器，最终传给Configuration对象的setTemplateLoader(TemplateLoader loader)方法。如从/user/data/templates和/temp/templates两个地方加载模板可以这么做：
	```java
	FileTemplateLoader ftl1 = new FileTemplateLoader(new File("/temp/templates"));  
	FileTemplateLoader ftl2 = new FileTemplateLoader(new File("/user/data/templates"));  
	ClassTemplateLoader ctl1 = new ClassTemplateLoader(TemplateLoaderTest.class, "/");  
	TemplateLoader[] loaders = new TemplateLoader[]{ftl1, ftl2, ctl1};  
	MultiTemplateLoader mtl = new MultiTemplateLoader(loaders);  
	cfg.setTemplateLoader(mtl);  
	```
	现在，FreeMarker将会尝试从 /tmp/templates目录加载模板，如果在这个目录下没有发现请求的模板，它就会继续尝试从/usr/data/templates 目录下加载，如果还是没有发现请求的模板， 那么它就会使用类加载器来加载模板。

 - 从其他资源加载模板
 如果内建的类加载器都不适合使用，那么就需要来编写自己的类加载器了，这个类需要实现freemarker.cache.TemplateLoader接口，然后将它传递给Configuration对象的setTemplateLoader(TemplateLoader loader)方法。可以阅读API JavaDoc文档获取更多信息。如果模板需要通过URL访问其他模板，那么就不需要实现TemplateLoader接口了，可以选择子接口freemarker.cache.URLTemplateLoader来替代，只需实现URLgetURL(String templateName)方法即可。

完整的FreeMaker学习资料：[FreeMarker_2.3.23_Manual_zh_CN][1]


  [1]: https://github.com/yuchengxin/FreeMarker_2.3.23_Manual_zh_CN.git
+++
title = "Sping in my view"
date = 2020-02-14

[taxonomies]
categories = ["技术"]
tags = ["spring","技术"]
+++

spring正统治者java web开发界，任何一个java web开发者都绕不开spring。搜一下网络，看一眼官网，密密麻麻的spring相关的资料引入眼帘，让人丈二和尚摸不着头脑。我现在依然能够想起曾经在项目上第一次接触spring时那个无知和茫然。对任何spring新手来讲，需要面对的第一个问题便是：**spring到底是什么？**

<!-- more -->
***

## spring的含义
回答一下spring是什么？
我认为spring有以下2类含义：

1. 狭义的spring框架，也就是最开始仅支持aop/ioc的依赖注入框架。这是spring的核心。
2. 广义的spring生态，围绕spring框架的一系列开发组件，在spring框架为基础的开发模式下，涵盖了在web领域的方方面面，能够体现web领域的主流的技术和思想变化。如spring-boot、spring-data、spring-cloud、spring-security等等。
&emsp;
***

## spring的发展史
spring这个词从框架到生态，这其中的发展史可以一栏web的变化。

spring框架起源于当时介绍依赖注入的书籍《Expert One-on-One J2EE设计和开发》，

在后来的web潮流中随着xml的流行增加xml的支持，

也随着对配置简化和快速构建项目的需求出现了java配置和spring boot，

在微服务流行之际出现了spring cloud，

在如今响应式编程有些抬头的时候也不忘推出WebFlux来支持。

**spring生态的发展史，几乎可以说是大家对web领域的探究。**
&emsp;
***


## spring的结构
spring的生态中，以spring框架为基础，遵循某类设计的一系列组件。而我们在使用spring生态内的东西的时候，也是会按照某类开发模式（pattern，一个以spring框架为基础构建的pattern）进行的。

### **metadata/logic/data的pattern**
这个模式也可以名为metadata（context）/logic/data，
本质上是一类事物。
* metadata指的是当前环境的信息，也是程序中的元信息，根据环境而变化。
* logic指的是程序中的逻辑，反映了程序的业务/执行逻辑，根据需求变化而变化。
* status值得是程序中的存储数据/状态，反映了反映的是业务用户的信息，根据用户变化而变化。

### spring结构对pattern的对照
按照上面的pattern的划分，我们可以对应到常见的spring的web程序结构中来：
1. metadata:一系列properties/yaml，
2. logic:一系列controller/service/repo，
3. status:一系列数据库/缓存数据。

一个新API，
* 如果需要读取新环境数据，需要properties的变化，并把这个变量值依赖注入到bean的计算逻辑中。
* 需要新逻辑需要计算中改变那些controller/service/repo的部分逻辑；
* 需要引用现有的数据辅助计算，需要读取数据库/缓存，并反映到controller/service/repo的计算逻辑中。
&emsp;
***

## 开发者在spring中对logic和metadata的使用
而对开发者来讲，一般能修改的便是logic和metedata
### **logic借助依赖注入表达依赖关系**
logic的使用依靠bean与bean之间的引用（依赖注入）。

spring框架之中，代码（或者说类/组件）与代码的交互是依靠bean来统一管理的。
bean想要引用其他bean必须依靠依赖注入来表达。


拿常见的web开发下的mvc模式来说，Controller一类的bean里必须要引用Service一类的bean才能使用。
同时，bean之间的引用存在一定的依赖关系，因此在bean的依赖注入（autowired）时需要注意这种顺序。


### **metadata导入**
元数据，也可以说是配置，依赖于外部环境的一些数据，靠依赖注入的方式注入到bean，配置不同环境下的不同bean。在properties或者yaml文件中定义元数据（配置）的具体值。


比如，拿常见的spring的jdbc初始化来讲。
单数据源下的jdbc初始化，一般采用默认配置，所以我们只需要在properties或者yaml文件中配置环境变量即可。可以从多数据源下，一窥spring中存在依赖关系引用的bean之间的使用。[参考代码](https://stackoverflow.com/questions/30362546/how-to-use-2-or-more-databases-with-spring)

&emsp;
***


## spring中组件的初始化顺序
为了满足status的使用，spring会在程序可以真正工作前，完成对metadata（properties）和logic（bean）的初始化，对外只会对用户开放stauts的变化。因此，而我们可以看一眼spring的初始化顺序，对properties/bean的加载和使用。

spring应用中，应用的所有信息都保存在applicationContext，spring初始化也就是从applicationcontext开始初始化的。最终会调用[refresh函数](https://github.com/spring-projects/spring-framework/blob/3a0f309e2c9fdbbf7fb2d348be861528177f8555/spring-context/src/main/java/org/springframework/context/support/AbstractApplicationContext.java#L515-L577)


```java
@Override
 public void refresh() throws BeansException, IllegalStateException {
  synchronized (this.startupShutdownMonitor) {
   // Prepare this context for refreshing.（包括properties或者yaml的环境变量，以及在设置pre-refresh的监听器）
   prepareRefresh();

   // Tell the subclass to refresh the internal bean factory.（获得能够创建bean的工厂，工厂模式）
   ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

   // Prepare the bean factory for use in this context.
   prepareBeanFactory(beanFactory);（准备bean工厂相关的上下文，得到bean的原始信息，扫描bean，准备EventPublisher，ResourceLoader等等都在这里，常见的scan一类注解在这里起作用）

   try {
    // Allows post-processing of the bean factory in context subclasses.（注册bean工厂相关的post-processing的processor，在工厂创建完成之后）
    postProcessBeanFactory(beanFactory);

    // Invoke factory processors registered as beans in the context.（执行bean工厂相关的post-processing的processor，在工厂创建完成之后）
    invokeBeanFactoryPostProcessors(beanFactory);

    // Register bean processors that intercept bean creation.（在bean工厂中注册bean创建完成后的postProcessor）
    registerBeanPostProcessors(beanFactory);

    // Initialize message source for this context.（为应用上下文准备数据源/消息源）
    initMessageSource();

    // Initialize event multicaster for this context.（准备应用事件的广播器）
    initApplicationEventMulticaster();

    // Initialize other special beans in specific context subclasses.（初始化其他特殊bean）
    onRefresh();

    // Check for listener beans and register them.（注册和检查listener相关的bean）
    registerListeners();

    // Instantiate all remaining (non-lazy-init) singletons.（初始化所有non-lazy-init的bean，non-lazy-init指非延迟初始化的bean）
    finishBeanFactoryInitialization(beanFactory);

    // Last step: publish corresponding event.（发布spring的初始化完成event）
    finishRefresh();
   }

   catch (BeansException ex) {
    if (logger.isWarnEnabled()) {
     logger.warn("Exception encountered during context initialization - " +
       "cancelling refresh attempt: " + ex);
    }

    // Destroy already created singletons to avoid dangling resources.
    destroyBeans();

    // Reset 'active' flag.
    cancelRefresh(ex);

    // Propagate exception to caller.
    throw ex;
   }

   finally {
    // Reset common introspection caches in Spring's core, since we
    // might not ever need metadata for singleton beans anymore...
    resetCommonCaches();
   }
  }
 }
```
建立在这个部分代码之上的，便是spring的众多组件围绕的大生态。
如spring-boot，也会利用其中的机制形成一个新组件，满足人们对快速开发的需求。
[参考代码](https://www.jianshu.com/p/8f3b9af9bb33)


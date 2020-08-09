title: Spring Bean 生命周期
author: wen
tags: []
categories:
  - Spring
  - SpringFramework
date: 2020-08-09 20:38:00
---

# [spring IOC容器](/2020/08/08/spring源码（一）/)
# [spring源码编译](/2020/08/07/spring源码编译/)

# Spring容器是
* 从概念上讲：Spring 容器是 Spring 框架的核心，是用来管理对象的。容器将创建对象，把它们连接在一起，配置它们，并管理他们的整个生命周期从创建到销毁。
* 从具象化讲：在java项目中，我们使用实现了org.springframework.context.ApplicationContext接口的实现类。
  [spring容器子类](/2020/08/08/spring源码（一）/#spring容器)
* 从代码上讲：一个Spring容器就是某个实现了ApplicationContext接口的类的实例。也就是说，从代码层面，Spring容器其实就是一个

## BeanFactory
BeanFactory 是 Spring 的“心脏”。它就是 Spring IoC 容器的真面目。Spring 使用 BeanFactory 来实例化、配置和管理 Bean。

BeanFactory：是IOC容器的核心接口， 它定义了IOC的基本功能，我们看到它主要定义了getBean方法。getBean方法是IOC容器获取bean对象和引发依赖注入的起点。方法的功能是返回特定的名称的Bean。

BeanFactory 是初始化 Bean 和调用它们生命周期方法的“吃苦耐劳者”。注意，BeanFactory 只能管理单例（Singleton）Bean 的生命周期。它不能管理原型(prototype,非单例)Bean 的生命周期。这是因为原型 Bean 实例被创建之后便被传给了客户端,容器失去了对它们的引用。
DefaultListableBeanFactory类图：
![DefaultListableBeanFactory类图](/Spring-Bean-生命周期/DefaultListableBeanFactory.png)

*  1、BeanFactory作为一个主接口不继承任何接口，暂且称为一级接口。
是Spring bean容器的根接口，提供获取bean，是否包含bean,是否单例与原型，获取bean类型，bean 别名的方法 。它最主要的方法就是getBean(String beanName)。
*  2、有3个子接口继承了它，进行功能上的增强。这3个子接口称为二级接口。
 * HierarchicalBeanFactory：提供父容器的访问功能
 
 * ListableBeanFactory：提供了批量获取Bean的方法
 
 * AutowireCapableBeanFactory：在BeanFactory基础上实现对已存在实例的管理


* 3、ConfigurableBeanFactory可以被称为三级接口，对二级接口HierarchicalBeanFactory进行了再次增强，它还继承了另一个外来的接口SingletonBeanRegistry
主要单例bean的注册，生成实例，以及统计单例bean

* 4、ConfigurableListableBeanFactory是一个更强大的接口，继承了上述的所有接口，无所不包，称为四级接口。　（这4级接口是BeanFactory的基本接口体系。继续，下面是继承关系的2个抽象类和2个实现类：）
继承了上述的所有接口，增加了其他功能：比如类加载器,类型转化,属性编辑器,BeanPostProcessor,作用域,bean定义,处理bean依赖关系, bean如何销毁…
* 5、AbstractBeanFactory作为一个抽象类，实现了三级接口ConfigurableBeanFactory大部分功能。

* 6、AbstractAutowireCapableBeanFactory同样是抽象类，继承自AbstractBeanFactory，并额外实现了二级接口AutowireCapableBeanFactory

* 7、DefaultListableBeanFactory继承自AbstractAutowireCapableBeanFactory，实现了最强大的四级接口ConfigurableListableBeanFactory，并实现了一个外来接口BeanDefinitionRegistry，它并非抽象类。

  实现了ConfigurableListableBeanFactory，实现上述BeanFactory所有功能。它还可以注册BeanDefinition
BeanFactory的源码： 
```
https://docs.spring.io/spring/docs/5.1.18.BUILD-SNAPSHOT/javadoc-api/

package org.springframework.beans.factory;

import org.springframework.beans.BeansException;
import org.springframework.core.ResolvableType;
import org.springframework.lang.Nullable;

public interface BeanFactory {

	/**
	 * 用来引用一个实例，或把它和工厂产生的Bean区分开，就是说，如果一个FactoryBean的名字为a，那么，&a会得到那个Factory
	 */
	String FACTORY_BEAN_PREFIX = "&";
	/*
	 * 五个不同形式的getBean方法，获取实例
	 */
	Object getBean(String name) throws BeansException;
	<T> T getBean(String name, Class<T> requiredType) throws BeansException;
	Object getBean(String name, Object... args) throws BeansException;
	<T> T getBean(Class<T> requiredType) throws BeansException;
	<T> T getBean(Class<T> requiredType, Object... args) throws BeansException;

	//2个不同形式 返回指定bean的提供程序，以允许按需延迟检索实例，包括可用性和唯一性选项。
	<T> ObjectProvider<T> getBeanProvider(Class<T> requiredType);
	<T> ObjectProvider<T> getBeanProvider(ResolvableType requiredType);

	/**
	 *  bean 是否存在
	 */
	boolean containsBean(String name);

	// 是否为单实例
	boolean isSingleton(String name) throws NoSuchBeanDefinitionException;

	// 是否为原型（多实例）
	boolean isPrototype(String name) throws NoSuchBeanDefinitionException;

	// 2个不同形式 名称、类型是否匹配
	boolean isTypeMatch(String name, ResolvableType typeToMatch) throws NoSuchBeanDefinitionException;
	boolean isTypeMatch(String name, Class<?> typeToMatch) throws NoSuchBeanDefinitionException;

	 // 2个不同形式 获取类型
	@Nullable
	Class<?> getType(String name) throws NoSuchBeanDefinitionException;
	@Nullable
	Class<?> getType(String name, boolean allowFactoryBeanInit) throws NoSuchBeanDefinitionException;

	// 根据实例的名字获取实例的别名
	String[] getAliases(String name);

}

```
具体：
　　1、5个获取实例的方法。getBean的重载方法。
　　2、7个判断的方法。判断是否存在，是否为单例、原型，名称类型是否匹配。
　　3、1个获取类型的方法、一个获取别名的方法。根据名称获取类型、根据名称获取别名。一目了然！
总结：
　　这13个方法，很明显，这是一个典型的工厂模式的工厂接口。
AnnotationConfigApplicationContext 例子
```
从一个或多个
基于Java的配置类中加载Spring应用上下文

package com.yanwen.springTest;

import com.yanwen.springTest.service.MemberService;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.Configuration;

@ComponentScan("com.yanwen.springTest")
@Configuration
public class AppConfig {
   public static void main(String[] args){
      AnnotationConfigApplicationContext annotationConfigApplicationContext = new AnnotationConfigApplicationContext(AppConfig.class);
      System.out.println(annotationConfigApplicationContext.getBean(MemberService.class));
    }
}

package com.yanwen.springTest.service;

import org.springframework.stereotype.Service;

@Service
public class MemberService {
}

```
1. 从一个或多个基于Java的配置类中加载Spring应用上下文（AnnotationConfigApplicationContext）。
2. 通过BeanFactory启动IoC容器时，并不会初始化配置文件中定义的Bean，初始化动作发生在第一个调用时。
3. 对于单实例（singleton）的Bean来说，BeanFactory会缓存Bean实例，所以第二次使用getBean时直接从IoC容器缓存中获取Bean。

## ApplicationContext
如果说BeanFactory是Spring的心脏，那么ApplicationContext就是完整的躯体了，ApplicationContext由BeanFactory派生而来，提供了更多面向实际应用的功能。

BeanFactorty接口提供了配置框架及基本功能，但是无法支持spring的aop功能和web应用。而ApplicationContext接口作为BeanFactory的派生，因而提供BeanFactory所有的功能。而且ApplicationContext还在功能上做了扩展，相较于BeanFactorty，ApplicationContext还提供了以下的功能： 

* （1）MessageSource, 提供国际化的消息访问  
* （2）资源访问，如URL和文件  
* （3）事件传播特性，即支持aop特性
* （4）载入多个（有继承关系）上下文 ，使得每一个上下文都专注于一个特定的层次，比如应用的web层 

ApplicationContext：是IOC容器另一个重要接口， 它继承了BeanFactory的基本功能， 同时也继承了容器的高级功能，如：MessageSource（国际化资源接口）、ResourceLoader（资源加载接口）、ApplicationEventPublisher（应用事件发布接口）等。


### ApplicationContext常用实现类	作用

* AnnotationConfigApplicationContext	从一个或多个基于java的配置类中加载上下文定义，适用于java注解的方式。
* ClassPathXmlApplicationContext	从类路径下的一个或多个xml配置文件中加载上下文定义，适用于xml配置的方式。
* FileSystemXmlApplicationContext	从文件系统下的一个或多个xml配置文件中加载上下文定义，也就是说系统盘符中加载xml配置文件。
* AnnotationConfigWebApplicationContext	专门为web应用准备的，适用于注解方式。
* XmlWebApplicationContext	从web应用下的一个或多个xml配置文件加载上下文定义，适用于xml配置方式。

### 三种装配机制：

* 在XMl中进行显示配置
* 在Java中进行显示配置
* 隐式的bean发现机制和自动装配
	* 组件扫描（component scanning）：Spring会自动发现应用上下文中所创建的bean。
	* 自动装配（autowiring）：Spring自动满足bean之间的依赖。
    
（使用的优先性: 3>2>1）尽可能地使用自动配置的机制，显示配置越少越好。当必须使用显示配置bean的时候（如：有些源码不是由你来维护的，而当你需要为这些代码配置bean的时候），推荐使用类型安全比XML更加强大的JavaConfig。最后只有当你想要使用便利的XML命名空间，并且在JavaConfig中没有同样的实现时，才使用XML。
代码示例：

通过xml文件将配置加载到IOC容器中
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
     <!--若没写id，则默认为com.test.Man#0,#0为一个计数形式-->
    <bean id="man" class="com.test.Man"></bean>
</beans>
```
```
public class Test {
    public static void main(String[] args) {
        //加载项目中的spring配置文件到容器
        //ApplicationContext context = new ClassPathXmlApplicationContext("resouces/applicationContext.xml");
        //加载系统盘中的配置文件到容器
        ApplicationContext context = new FileSystemXmlApplicationContext("E:/Spring/applicationContext.xml");
        //从容器中获取对象实例
        Man man = context.getBean(Man.class);
        man.driveCar();
    }
}
```
通过java注解的方式将配置加载到IOC容器
```
//同xml一样描述bean以及bean之间的依赖关系
@Configuration
public class ManConfig {
    @Bean
    public Man man() {
        return new Man(car());
    }
    @Bean
    public Car car() {
        return new QQCar();
    }
}
public class Test {
    public static void main(String[] args) {
        //从java注解的配置中加载配置到容器
        ApplicationContext context = new AnnotationConfigApplicationContext(ManConfig.class);
        //从容器中获取对象实例
        Man man = context.getBean(Man.class);
        man.driveCar();
    }
}
```
隐式的bean发现机制和自动装配
```
/**
 * 这是一个游戏光盘的实现
 */
//这个简单的注解表明该类回作为组件类，并告知Spring要为这个创建bean。
@Component
public class GameDisc implements Disc{
    @Override
    public void play() {
        System.out.println("我是马里奥游戏光盘。");
    }
}
```
不过，组件扫描默认是不启用的。我们还需要显示配置一下Spring，从而命令它去寻找@Component注解的类，并为其创建bean。
```
@Configuration
@ComponentScan
public class DiscConfig {
}
```
我们在DiscConfig上加了一个@ComponentScan注解表示在Spring中开启了组件扫描，默认扫描与配置类相同的包，就可以扫描到这个GameDisc的Bean了。这就是Spring的自动装配机制。

由于ApplicationContext会预先初始化所有的Singleton Bean，于是在系统创建前期会有较大的系统开销，但一旦ApplicationContext初始化完成，程序后面获取Singleton Bean实例时候将有较好的性能。也可以为bean设置lazy-init属性为true，即Spring容器将不会预先初始化该bean。

### AnnotationConfigApplicationContext类图：
![AnnotationConfigApplicationContext类图](/Spring-Bean-生命周期/AnnotationConfigApplicationContext.png)


# 如何产生bean

Bean factory implementations should support the standard bean lifecycle interfaces as far as possible. The full set of initialization methods and their standard order is:

BeanNameAware's setBeanName
BeanClassLoaderAware's setBeanClassLoader
BeanFactoryAware's setBeanFactory
EnvironmentAware's setEnvironment
EmbeddedValueResolverAware's setEmbeddedValueResolver
ResourceLoaderAware's setResourceLoader (only applicable when running in an application context)
ApplicationEventPublisherAware's setApplicationEventPublisher (only applicable when running in an application context)
MessageSourceAware's setMessageSource (only applicable when running in an application context)
ApplicationContextAware's setApplicationContext (only applicable when running in an application context)
ServletContextAware's setServletContext (only applicable when running in a web application context)
postProcessBeforeInitialization methods of BeanPostProcessors
InitializingBean's afterPropertiesSet
a custom init-method definition
postProcessAfterInitialization methods of BeanPostProcessors

On shutdown of a bean factory, the following lifecycle methods apply:

postProcessBeforeDestruction methods of DestructionAwareBeanPostProcessors
DisposableBean's destroy
a custom destroy-method definition

## 循环依赖



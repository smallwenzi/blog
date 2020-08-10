title: Spring Bean 生命周期
author: wen
tags: []
categories:
  - Spring
  - SpringFramework
date: 2020-08-09 20:38:00
---

# 如何产生bean

```
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
[如何在spring源码中新建model及创建上面的代码](/2020/08/07/spring源码编译/#新建Model)
[关于ApplicationContext](/2020/08/08/spring源码（一）/#ApplicationContext)
以上代码：
* 从一个Java的配置类中加载Spring应用上下文（AnnotationConfigApplicationContext）。
* 通过BeanFactory启动IoC容器时，并不会初始化配置文件中定义的Bean，初始化动作发生在第一个调用时。
* 对于单实例（singleton）的Bean来说，BeanFactory会缓存Bean实例，所以第二次使用getBean时直接从IoC容器缓存中获取Bean。

# 启动过程分析
记住，一定要在电脑中打开源码，不然纯看是很累的。
## AnnotationConfigApplicationContext构造方法
```
public class AnnotationConfigApplicationContext extends GenericApplicationContext implements AnnotationConfigRegistry {

	private final AnnotatedBeanDefinitionReader reader;

	private final ClassPathBeanDefinitionScanner scanner;
    
	public AnnotationConfigApplicationContext() {
    ////调用父类GenericApplicationContext无参构造函数，初始化一个BeanFactory: DefaultListableBeanFactory beanFactory = new DefaultListableBeanFactory()
    
     //在IOC容器中初始化一个 注解bean读取器AnnotatedBeanDefinitionReader
		this.reader = new AnnotatedBeanDefinitionReader(this);
        //在IOC容器中初始化一个 按类路径扫描注解bean的 扫描器
		this.scanner = new ClassPathBeanDefinitionScanner(this);
	}
    /**
     AnnotationConfigApplicationContext annotationConfigApplicationContext = new AnnotationConfigApplicationContext(AppConfig.class);
    这句代码调用以下方法
  */
	public AnnotationConfigApplicationContext(Class<?>... componentClasses) {
    // 初始化bean读取器和扫描器;
		this();
		register(componentClasses);
		refresh();
	}
    
}
```
### GenericApplicationContext 构造方法
```
public class GenericApplicationContext extends AbstractApplicationContext implements BeanDefinitionRegistry {
 	private final DefaultListableBeanFactory beanFactory;
    public GenericApplicationContext() {
    //初始化一个BeanFactory
		this.beanFactory = new DefaultListableBeanFactory();
	}

}
```
### DefaultListableBeanFactory 构造方法
[DefaultListableBeanFactory类图](/2020/08/08/spring源码（一）/#BeanFactory)


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
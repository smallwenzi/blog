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
[spring源码注释](https://github.com/smallwenzi/springframeworkcode)

Spring bean 默认是单例
为了弄清楚Bean是怎么来的，花费了大把功夫，现在要把Bean Definition的加载、解析、处理、注册到bean工厂的过程记下来

## AnnotationConfigApplicationContext构造方法
```
public class AnnotationConfigApplicationContext extends GenericApplicationContext implements AnnotationConfigRegistry {

	private final AnnotatedBeanDefinitionReader reader;

	private final ClassPathBeanDefinitionScanner scanner;
    
    /**
     AnnotationConfigApplicationContext annotationConfigApplicationContext = new AnnotationConfigApplicationContext(AppConfig.class);
    这句代码调用以下方法
   根据参数类型可以知道，其实可以传入多个annotatedClasses，但是这种情况出现的比较少
  */
	public AnnotationConfigApplicationContext(Class<?>... componentClasses) {
     //详细看AnnotationConfigApplicationContext 无参构造
		this();
		register(componentClasses);
		refresh();
	}
    
   public AnnotationConfigApplicationContext() {
    ////调用父类GenericApplicationContext无参构造函数，初始化一个BeanFactory: DefaultListableBeanFactory beanFactory = new DefaultListableBeanFactory()
     //在IOC容器中初始化一个 注解bean读取器AnnotatedBeanDefinitionReader
		this.reader = new AnnotatedBeanDefinitionReader(this);
        //在IOC容器中初始化一个 按类路径扫描注解bean的 扫描器
		this.scanner = new ClassPathBeanDefinitionScanner(this);
	}
    
}
```
* 调用无参构造函数，会先调用父类GenericApplicationContext的构造函数
	* 父类的构造函数里面就是初始化DefaultListableBeanFactory，并且赋值给beanFactory
   * 本类的构造函数里面，初始化了一个读取器：AnnotatedBeanDefinitionReader read，一个扫描器ClassPathBeanDefinitionScanner scanner
	* scanner的用处不是很大，它仅仅是在我们外部手动调用 .scan 等方法才有用，常规方式是不会用到scanner对象的
* 
* 

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
#### DefaultListableBeanFactory 构造方法

默认实现了ListableBeanFactory和BeanDefinitionRegistry接口，基于bean definition对象，是一个成熟的bean factroy。

最典型的应用是：在访问bean前，先注册所有的definition（可能从bean definition配置文件中）。使用预先建立的bean定义元数据对象，从本地的bean definition表中查询bean definition因而将不会花费太多成本。

DefaultListableBeanFactory既可以作为一个单独的beanFactory，也可以作为自定义beanFactory的父类。
![DefaultListableBeanFactory类图](Spring-Bean-生命周期/DefaultListableBeanFactory.png)

```
public class DefaultListableBeanFactory extends AbstractAutowireCapableBeanFactory
        implements ConfigurableListableBeanFactory, BeanDefinitionRegistry, Serializable {
        //存储BeanDefinition对象
	private final Map<String, BeanDefinition> beanDefinitionMap = new ConcurrentHashMap<>(256);
    
   	/** 存储 spring 单例对象  */
	private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256);
	/**
	 * Create a new DefaultListableBeanFactory.
	 */
	public DefaultListableBeanFactory() {
		super();
	}
 }
```
##### AbstractAutowireCapableBeanFactory构造方法
```
	super();
    //自动装配时忽略指定接口或类的依赖注入
		ignoreDependencyInterface(BeanNameAware.class);
		ignoreDependencyInterface(BeanFactoryAware.class);
		ignoreDependencyInterface(BeanClassLoaderAware.class);
```
###### 	AbstractBeanFactory
```
/**
	 * Create a new AbstractBeanFactory.
	 */
	public AbstractBeanFactory() {
	}
 ```
### //初始化一个Bean读取器
```
this.reader = new AnnotatedBeanDefinitionReader(this);
```
#### AnnotatedBeanDefinitionReader构造方法
```
	public AnnotatedBeanDefinitionReader(BeanDefinitionRegistry registry) {
		//这里的BeanDefinitionRegistry当然就是AnnotationConfigApplicationContext的实例了
		this(registry, getOrCreateEnvironment(registry));
	}
```
AnnotatedBeanDefinitionReader 方法
```
	public AnnotatedBeanDefinitionReader (BeanDefinitionRegistry registry, Environment environment) {
		Assert.notNull(registry, "BeanDefinitionRegistry must not be null");
		Assert.notNull(environment, "Environment must not be null");
		this.registry = registry;
		this.conditionEvaluator = new ConditionEvaluator(registry, environment, null);
		//让我们把目光移动到这个方法的最后一行，进入registerAnnotationConfigProcessors方法：
 this.registry 是    AnnotationConfigApplicationContext的实例了   AnnotationConfigUtils.registerAnnotationConfigProcessors(this.registry);
	}
```
#### AnnotationConfigUtils.registerAnnotationConfigProcessors
```
	public static void registerAnnotationConfigProcessors(BeanDefinitionRegistry registry) {
		registerAnnotationConfigProcessors(registry, null);
	}
```
在看registerAnnotationConfigProcessors方法
* 这里会一连串注册好几个Bean，在这其中最重要的一个Bean（没有之一）就是BeanDefinitionRegistryPostProcessor Bean。
		* ConfigurationClassPostProcessor实现BeanDefinitionRegistryPostProcessor接口，
		* BeanDefinitionRegistryPostProcessor接口又扩展了BeanFactoryPostProcessor接口，
		* BeanFactoryPostProcessor是Spring的扩展点之一，
		*  ConfigurationClassPostProcessor是Spring极为重要的一个类，必须牢牢的记住上面所说的这个类和它的继承关系。
		* 除了注册了ConfigurationClassPostProcessor，还注册了其他Bean，其他Bean也都实现了其他接口，比如BeanPostProcessor等。
		* BeanPostProcessor接口也是Spring的扩展点之一。
至此，实例化AnnotatedBeanDefinitionReader reader分析完毕。
```
// this.registry 是    AnnotationConfigApplicationContext的实例了   
	public static Set<BeanDefinitionHolder> registerAnnotationConfigProcessors(
			BeanDefinitionRegistry registry, @Nullable Object source) {
           //这里返回AnnotationConfigApplicationContext对象中DefaultListableBeanFactory类实例
		DefaultListableBeanFactory beanFactory = unwrapDefaultListableBeanFactory(registry);
BeanDefinition		if (beanFactory != null) {

			if (!(beanFactory.getDependencyComparator() instanceof AnnotationAwareOrderComparator)) {
				//用来支持Spring的Ordered类、@Order注解和@Priority注解。
                beanFactory.setDependencyComparator(AnnotationAwareOrderComparator.INSTANCE);
			}
	
			if (!(beanFactory.getAutowireCandidateResolver() instanceof ContextAnnotationAutowireCandidateResolver)) {
				beanFactory.setAutowireCandidateResolver(new ContextAnnotationAutowireCandidateResolver());
			}
		}

		Set<BeanDefinitionHolder> beanDefs = new LinkedHashSet<>(8);

		//判断容器中是否已经存在了ConfigurationClassPostProcessor Bean
		if (!registry.containsBeanDefinition(CONFIGURATION_ANNOTATION_PROCESSOR_BEAN_NAME)) {
			//如果不存在（当然这里肯定是不存在的），就通过RootBeanDefinition的构造方法获得ConfigurationClassPostProcessor的BeanDefinition，RootBeanDefinition是BeanDefinition的子类
            
			RootBeanDefinition def = new RootBeanDefinition(ConfigurationClassPostProcessor.class);
			def.setSource(source);
			beanDefs.add(registerPostProcessor(registry, def, CONFIGURATION_ANNOTATION_PROCESSOR_BEAN_NAME));
		}

		if (!registry.containsBeanDefinition(AUTOWIRED_ANNOTATION_PROCESSOR_BEAN_NAME)) {
			RootBeanDefinition def = new RootBeanDefinition(AutowiredAnnotationBeanPostProcessor.class);
			def.setSource(source);
			beanDefs.add(registerPostProcessor(registry, def, AUTOWIRED_ANNOTATION_PROCESSOR_BEAN_NAME));
		}

		// Check for JSR-250 support, and if present add the CommonAnnotationBeanPostProcessor.
		if (jsr250Present && !registry.containsBeanDefinition(COMMON_ANNOTATION_PROCESSOR_BEAN_NAME)) {
			RootBeanDefinition def = new RootBeanDefinition(CommonAnnotationBeanPostProcessor.class);
			def.setSource(source);
			beanDefs.add(registerPostProcessor(registry, def, COMMON_ANNOTATION_PROCESSOR_BEAN_NAME));
		}

		// Check for JPA support, and if present add the PersistenceAnnotationBeanPostProcessor.
		if (jpaPresent && !registry.containsBeanDefinition(PERSISTENCE_ANNOTATION_PROCESSOR_BEAN_NAME)) {
			RootBeanDefinition def = new RootBeanDefinition();
			try {
				def.setBeanClass(ClassUtils.forName(PERSISTENCE_ANNOTATION_PROCESSOR_CLASS_NAME,
						AnnotationConfigUtils.class.getClassLoader()));
			}
			catch (ClassNotFoundException ex) {
				throw new IllegalStateException(
						"Cannot load optional framework class: " + PERSISTENCE_ANNOTATION_PROCESSOR_CLASS_NAME, ex);
			}
			def.setSource(source);
			beanDefs.add(registerPostProcessor(registry, def, PERSISTENCE_ANNOTATION_PROCESSOR_BEAN_NAME));
		}

		if (!registry.containsBeanDefinition(EVENT_LISTENER_PROCESSOR_BEAN_NAME)) {
			RootBeanDefinition def = new RootBeanDefinition(EventListenerMethodProcessor.class);
			def.setSource(source);
			beanDefs.add(registerPostProcessor(registry, def, EVENT_LISTENER_PROCESSOR_BEAN_NAME));
		}

		if (!registry.containsBeanDefinition(EVENT_LISTENER_FACTORY_BEAN_NAME)) {
			RootBeanDefinition def = new RootBeanDefinition(DefaultEventListenerFactory.class);
			def.setSource(source);
			beanDefs.add(registerPostProcessor(registry, def, EVENT_LISTENER_FACTORY_BEAN_NAME));
		}

		return beanDefs;
	}
```
RootBeanDefinition 是BeanDefinition接口子类
 BeanDefinition是什么，顾名思义，它是用来描述Bean的，
	 *  里面存放着关于Bean的一系列信息，比如Bean的作用域，Bean所对应的Class，
	 *  是否懒加载，是否Primary等等，这个BeanDefinition也相当重要，
     
![RootBeanDefinition类图](Spring-Bean-生命周期/RootBeanDefinition.png)

#### registerPostProcessor 方法
registerPostProcessor方法内部就是注册Bean，
```
private static BeanDefinitionHolder registerPostProcessor(
			BeanDefinitionRegistry registry, RootBeanDefinition definition, String beanName) {
	    //为BeanDefinition设置了一个Role，ROLE_INFRASTRUCTURE代表这是spring内部的，并非用户定义的
		definition.setRole(BeanDefinition.ROLE_INFRASTRUCTURE);
		//BeanDefinitionRegistry是接口，实现类：AnnotationConfigApplicationContext 实现方法位于：GenericApplicationContext
		//方法实现this.beanFactory.registerBeanDefinition(beanName, beanDefinition);

		registry.registerBeanDefinition(beanName, definition);
		return new BeanDefinitionHolder(definition, beanName);
	}
```
### 	register(componentClasses);
*把传入的类进行注册，这里有两个情况，
* 传入传统的配置类
* 传入bean（虽然一般没有人会这么做
* 看到后面会知道spring把传统的带上@Configuration的配置类称之为FULL配置类，不带@Configuration的称之为Lite配置类
* 但是我们这里先把带上@Configuration的配置类称之为传统配置类，不带的称之为普通bean
```
@Override
//componentClasses= AppConfig.class
	public void register(Class<?>... componentClasses) {
		Assert.notEmpty(componentClasses, "At least one component class must be specified");
       //AnnotatedBeanDefinitionReader 类对象
		this.reader.register(componentClasses);
	}
    
```
* AnnotatedBeanDefinitionReader 类register 方法
```
//componentClasses= AppConfig.class
public void register (Class<?>... componentClasses) {
		for (Class<?> componentClass : componentClasses) {
			registerBean(componentClass);
		}
	}
   
	public void registerBean(Class<?> beanClass) {
    //beanClass= AppConfig.class
		doRegisterBean(beanClass, null, null, null, null);
	}
    
    
  ``` 
#### AnnotatedBeanDefinitionReader的方法doRegisterBean
/**
	1 通过AnnotatedGenericBeanDefinition的构造方法，
	 获得配置类的BeanDefinition，这里是不是似曾相似，
	 在注册ConfigurationClassPostProcessor类的时候，
	 也是通过构造方法去获得BeanDefinition的，只不过当时是通过RootBeanDefinition去获得，
	 现在是通过AnnotatedGenericBeanDefinition去获得。

	 2判断需不需要跳过注册，Spring中有一个@Condition注解，如果不满足条件，就会跳过这个类的注册。

	3 然后是解析作用域，如果没有设置的话，默认为单例。

	 4 获得BeanName。

	 5 解析通用注解，填充到AnnotatedGenericBeanDefinition，解析的注解为Lazy，Primary，DependsOn，Role，Description。

	 6 限定符处理，不是特指@Qualifier注解，也有可能是Primary，或者是Lazy，或者是其他（理论上是任何注解，这里没有判断注解的有效性）。

	 7 把AnnotatedGenericBeanDefinition数据结构和beanName封装到一个对象中（这个不是很重要，可以简单的理解为方便传参）。

	 8 注册，最终会调用DefaultListableBeanFactory中的registerBeanDefinition方法去注册：
	 */
  ```   
  //beanClass= AppConfig.class
	private <T> void doRegisterBean(Class<T> beanClass, @Nullable String name,
			@Nullable Class<? extends Annotation>[] qualifiers, @Nullable Supplier<T> supplier,
			@Nullable BeanDefinitionCustomizer[] customizers) {
        //AnnotatedGenericBeanDefinition可以理解为一种数据结构，是用来描述Bean的，这里的作用就是把传入的标记了注解的类
		//转为AnnotatedGenericBeanDefinition数据结构，里面有一个getMetadata方法，可以拿到类上的注解
		AnnotatedGenericBeanDefinition abd = new AnnotatedGenericBeanDefinition(beanClass);
		//判断是否需要跳过注解，spring中有一个@Condition注解，当不满足条件，这个bean就不会被解析
		if (this.conditionEvaluator.shouldSkip(abd.getMetadata())) {
			return;
		}

		abd.setInstanceSupplier(supplier);
		//解析bean的作用域，如果没有设置的话，默认为单例
		ScopeMetadata scopeMetadata = this.scopeMetadataResolver.resolveScopeMetadata(abd);
		abd.setScope(scopeMetadata.getScopeName());
		//获得beanName
		String beanName = (name != null ? name : this.beanNameGenerator.generateBeanName(abd, this.registry));
		//解析通用注解，填充到AnnotatedGenericBeanDefinition，解析的注解为Lazy，Primary，DependsOn，Role，Description
		AnnotationConfigUtils.processCommonDefinitionAnnotations(abd);
		//限定符处理，不是特指@Qualifier注解，也有可能是Primary,或者是Lazy，或者是其他（理论上是任何注解，这里没有判断注解的有效性），如果我们在外面，以类似这种
		//AnnotationConfigApplicationContext annotationConfigApplicationContext = new AnnotationConfigApplicationContext(Appconfig.class);常规方式去初始化spring，
		//qualifiers永远都是空的，包括上面的name和instanceSupplier都是同样的道理
		//但是spring提供了其他方式去注册bean，就可能会传入了
		if (qualifiers != null) {
			//可以传入qualifier数组，所以需要循环处理
			for (Class<? extends Annotation> qualifier : qualifiers) {

				//Primary注解优先
				if (Primary.class == qualifier) {
					abd.setPrimary(true);
				}
				//Lazy注解
				else if (Lazy.class == qualifier) {
					abd.setLazyInit(true);
				}
				else { //其他，AnnotatedGenericBeanDefinition有个Map<String,AutowireCandidateQualifier>属性，直接push进去
					abd.addQualifier(new AutowireCandidateQualifier(qualifier));
				}
			}
		}
		if (customizers != null) {
			for (BeanDefinitionCustomizer customizer : customizers) {
				customizer.customize(abd);
			}
		}
		//这个方法用处不大，就是把AnnotatedGenericBeanDefinition数据结构和beanName封装到一个对象中
		BeanDefinitionHolder definitionHolder = new BeanDefinitionHolder(abd, beanName);
		definitionHolder = AnnotationConfigUtils.applyScopedProxyMode(scopeMetadata, definitionHolder, this.registry);
		//注册，最终会调用DefaultListableBeanFactory中的registerBeanDefinition方法去注册，
		//DefaultListableBeanFactory维护着一系列信息，比如beanDefinitionNames，beanDefinitionMap
		//beanDefinitionNames是一个List<String>,用来保存beanName
		//beanDefinitionMap是一个Map,用来保存beanName和beanDefinition
		BeanDefinitionReaderUtils.registerBeanDefinition(definitionHolder, this.registry);
	}

```
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
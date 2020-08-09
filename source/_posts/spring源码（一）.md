title: springFramework源码--IOC 容器
author: wen
tags:
  - IOC容器
  - 依赖注入
  - DI
  - Inversion of Control
categories:
  - spring
  - SpringFramework
date: 2020-08-08 14:45:00
---
Spring framework是一个Spring的基础框架，提供了Spring 的核心功能，比如依赖注入、事务管理、面向方面编程等等

IOC容器（ Inversion of Control ）是Spring Framework 最核心部分，它管理着Spring应用中bean的创建、 配置和管理，也叫依赖注入（dependency injection ）

# 什么是IOC容器
## 没有IOC容器
按照传统的做法，每个对象负责管理与自己相互协作的对象 
（即它所依赖的对象）的引用，这将会导致高度耦合和难以测试的代 码。

我们假定一个在线书店，通过BookService获取书籍：
```
public class BookService {
    private HikariConfig config = new HikariConfig();
    private DataSource dataSource = new HikariDataSource(config);

    public Book getBook(long bookId) {
        try (Connection conn = dataSource.getConnection()) {
            ...
            return book;
        }
    }
}
```

* 这种方式使用new的方式创建对象，
* 若测试BookService，是复杂的，因为必须要在真实的数据库环境下执行。

* 从上面的例子可以看出，如果一个系统有大量的组件，其生命周期和相互之间的依赖关系如果由组件自身来维护，不但大大增加了系统的复杂度，而且会导致组件之间极为紧密的耦合，继而给测试和维护带来了极大的困难。


## 有IOC容器
核心问题是：

* 谁负责创建组件？
* 谁负责根据依赖关系组装组件？
* 销毁时，如何按依赖顺序正确销毁？
解决这一问题的核心方案就是IoC。

传统的应用程序中，控制权在程序本身，程序的控制流程完全由开发者控制，例如：

在创建BookService的过程中，又创建了DataSource组件。这种模式的缺点是，一个组件如果要使用另一个组件，必须先知道如何正确地创建它。

在IoC模式下，控制权发生了反转，即从应用程序转移到了IoC容器，所有组件不再由应用程序自己创建和配置，而是由IoC容器负责，这样，应用程序只需要直接使用已经创建好并且配置好的组件。为了能让组件在IoC容器中被“装配”出来，需要某种“注入”机制，
例如，BookService自己并不会创建DataSource，而是等待外部通过setDataSource()方法来注入一个DataSource：

```
public class BookService { 
      private DataSource dataSource;
    public BookService(DataSource dataSource){
      this.DataSource=dataSource;
    }
}
```
不直接new一个DataSource，而是注入一个DataSource，这个小小的改动虽然简单，却带来了一系列好处：

* BookService不再关心如何创建DataSource，因此，不必编写读取数据库配置之类的代码；
* 测试BookService更容易，因为注入的是DataSource，可以使用内存数据库，而不是真实的MySQL配置。

因此，IoC又称为依赖注入（DI：Dependency Injection），它解决了一个最主要的问题：将组件的创建+配置与组件的使用相分离，并且，由IoC容器负责管理组件的生命周期。

因为IoC容器要负责实例化所有的组件，因此，有必要告诉容器如何创建组件，以及各组件的依赖关系.
以下这种通过在JavaConfig中进行显式配置bean。 
```
@Configuration
public Class JavaBean{

@Bean
 public HikariConfig hikariConfig(){
  return new HikariConfig();
 }
 @Bean
 public DataSource dataSource(HikariConfig config){
 return new HikariDataSource(config);
 }
 @Bean
 public BookService bookService(DataSource dataSource){
 return new BookService(dataSource);
 }
 
}

```

在Spring的IoC容器中，我们把所有组件统称为JavaBean，即配置一个组件就是配置一个Bean。

# 依赖注入的方式

* 通过构造方法注入

```
public class BookService { 
      private DataSource dataSource;
      @Autowired
    public BookService(DataSource dataSource){
      this.DataSource=dataSource;
    }
}
```
* 通过方法注入

```
public class BookService { 
      private DataSource dataSource;
      @Autowired
    public void setDataSource(DataSource dataSource){
      this.DataSource=dataSource;
    }
}
```
如果没有匹配的bean，那么在应用上下文创建的时候，Spring会抛出 
一个异常。为了避免异常的出现，你可以将@Autowired的 
required属性设置为false：
```
public class BookService { 
      private DataSource dataSource;
      @Autowired（required=false）
    public void setDataSource(DataSource dataSource){
      this.DataSource=dataSource;
    }
}
```
将required属性设置为false时，Spring会尝试执行自动装配，但 
是如果没有匹配的bean的话，Spring将会让这个bean处于未装配的状 
态。但是，把required属性设置为false时，你需要谨慎对待。如 
果在你的代码中没有进行null检查的话，这个处于未装配状态的属性 
有可能会出现NullPointerException。

如果有多个bean都能满足依赖关系的话，Spring将会抛出一个异常， 
表明没有明确指定要选择哪个bean进行自动装配

* 属性注入

```
public class BookService { 
 @Autowired
      private DataSource dataSource;
}
```

# spring容器
## AnnotationConfigApplicationContext
从一个或多个 
基于Java的配置类中加载Spring应用上下文。
你想从Java配置中加载应用上下文，那么可以使 
用AnnotationConfigApplicationContext
```
public class AppConfig {
   public static void main(String[] args){
      AnnotationConfigApplicationContext annotationConfigApplicationContext = new AnnotationConfigApplicationContext(AppConfig.class);
    }
}
```
应用上下文准备就绪之后，我们就可以调用上下文的getBean()方 
法从Spring容器中获取bean。
## AnnotationConfigWebApplicationContext
从一个或 
多个基于Java的配置类中加载Spring Web应用上下文。 
## ClassPathXmlApplicationContext
从类路径下的一个或 
多个XML配置文件中加载上下文定义，把应用上下文的定义文件 
作为类资源。 
## FileSystemXmlapplicationcontext
从文件系统下的一 
个或多个XML配置文件中加载上下文定义。 
## XmlWebApplicationContext：从Web应用下的一个或多个 
XML配置文件中加载上下文定义。

 

# javaConfig
java config是指基于java配置的spring。传统的Spring一般都是基本xml配置的，后来spring3.0新增了许多java config的注解，特别是spring boot，基本都是清一色的java config。
使用Annotation配合自动扫描能大幅简化Spring的配置
## @Configuration
在类上打上这一标签，表示这个类是配置类

# 常用注解

## @ComponentScan
该注解默认会扫描该类所在的包下所有的配置类，相当于之前的 <context:component-scan>。

* 指定包扫描,主要扫描该包下@Controller @Service @Respsitory @Component四个注解
```
@ComponentScan(value="com.songzixian")
```
* 指定排除要扫描的包
```
@ComponentScan(value="com.songzixian",excludeFilters ={@ComponentScan.Filter (type= FilterType.ANNOTATION,classes ={Component.class,Repository.class,Controller.class,Service.class})} )

```
* 指定要扫描的包
useDefaultFilters = false默认是true,需要改为false该类才生效
```
@ComponentScan(value="com.songzixian",includeFilters ={@ComponentScan.Filter (type= FilterType.ANNOTATION,classes ={Component.class,Repository.class,Controller.class,Service.class})},useDefaultFilters = false)
```

## @Bean
注解用于告诉方法，产生一个Bean对象，然后这个Bean对象交给Spring管理。产生这个Bean对象的方法Spring只会调用一次，随后这个Spring将会将这个Bean对象放在自己的IOC容器中

## @Autowired
 注解对自动装配何时何处被实现提供了更多细粒度的控制
 
 ## @Qualifier 
自动装配，如果容器中某个bean有多个实例，想要获取指定的的实例，可以用@Qualifier

## @scope
  用来配置 spring bean 的作用域，它标识 bean 的作用域。默认值是单例

  * 1、singleton:单例模式,全局有且仅有一个实例

  * 2、prototype:原型模式,每次获取Bean的时候会有一个新的实例

  * 3、request:request表示该针对每一次HTTP请求都会产生一个新的bean，同时该bean仅在当前HTTP request内有效

  * 4、session:session作用域表示该针对每一次HTTP请求都会产生一个新的bean，同时该bean仅在当前HTTP session内有效

  * 5、global session:只在portal应用中有用，给每一个 global http session 新建一个Bean实例。
  
## @Lazy
 容器启动不创建对象，调用的时候创建对象
 
## @Conditional（{WindowConditional.class}）
WindowConditional 实现condition接口，返回true

## @PostConstruct
初始化 注解在初始化的方法上面

##  @PreDestroy 
销毁 在容器销毁bean之前调用

## @Controller
标识一个该类是Spring MVC controller处理器，用来创建处理http请求的对象.

## @Service
用于标注业务层组件，说白了就是加入你有一个用注解的方式把这个类注入到spring配置中

##  @Resource
@Resource的作用相当于@Autowired
只不过@Autowired按byType自动注入，
而@Resource默认按 byName自动注入
@Resource有两个属性是比较重要的，分是name和type，Spring将@Resource注解的name属性解析为bean的名字，而type属性则解析为bean的类型。所以如果使用name属性，则使用byName的自动注入策略，而使用type属性时则使用byType自动注入策略。如果既不指定name也不指定type属性，这时将通过反射机制使用byName自动注入策略

## @Component
泛指组件，当组件不好归类的时候，我们可以使用这个注解进行标注

## @RestController
Spring4之后加入的注解，原来在@Controller中返回json需要@ResponseBody来配合，如果直接用@RestController替代@Controller就不需要再配置@ResponseBody，默认返回json格式。

## @RequestMapping
类定义处: 提供初步的请求映射信息，相对于 WEB 应用的根目录。
方法处: 提供进一步的细分映射信息，相对于类定义处的 URL。

## @RequestParam
用于将请求参数区数据映射到功能处理方法的参数上
```
@RequestMapping(value="", method=RequestMethod.POST)
    public String postUser(@RequestParam(value="phoneNum", required=true) String phoneNum ) String userName) {
        userService.create(phoneNum, userName);
        return "success";
    }
```
## @Repository
用于标注数据访问组件，即DAO组件

## @Required
适用于bean属性setter方法，并表示受影响的bean属性必须在XML配置文件在配置时进行填充。否则，容器会抛出一个BeanInitializationException异常。
 
 ## import
  ，@Import通过快速导入的方式实现把实例加入spring的IOC容器中
  * 1、直接填class数组方式
  ```
  @Import({ 类名.class , 类名.class... })
public class TestDemo {

}
  ```
  ## @PathVariable
  映射 URL 绑定的占位符
通过 @PathVariable 可以将 URL 中占位符参数绑定到控制器处理方法的入参中:URL 中的 {xxx} 占位符可以通过

@PathVariable("xxx") 绑定到操作方法的入参中
```
@GetMapping("/users/{username}")
public String userProfile(@PathVariable("username") String user) {
    return String.format("user %s", user);
}
```
 @PathVariable和@RequestParam的区别就在于：@RequestParam用来获得静态的URL请求参数；@PathVariable用来获得动态的URL请求入参
 
 ## @ResponseBody
 的作用其实是将java对象转为json格式的数据。
 ```
 @RequestMapping("/login.do")
@ResponseBody
public Object login(String name, String password, HttpSession session) {
	user = userService.checkLogin(name, password);
	session.setAttribute("user", user);
	return new JsonResult(user);
}
 ```
 ## @RequestBody
 是作用在形参列表上，用于将前台发送过来固定格式的数据【xml格式 或者 json等】封装为对应的 JavaBean 对象，
封装时使用到的一个对象是系统默认配置的 HttpMessageConverter进行解析，然后封装到形参上。
如上面的登录后台代码可以改为：
```
@RequestMapping("/login.do")
@ResponseBody
public Object login(@RequestBody User loginUuser, HttpSession session) {
	user = userService.checkLogin(loginUser);
	session.setAttribute("user", user);
	return new JsonResult(user);
}
```
## @PropertySource
目的是加载指定的属性文件
```
@PropertySource(value = {"classpath:people.properties"},ignoreResourceNotFound = false,encoding = "UTF-8",name = "people.properties")
public class PeopleProperties {
 
    private String name;
 
    private int age;
     public String getName() {
        return name;
    }
 
    public void setName(String name) {
        this.name = name;
    }
 
    public int getAge() {
        return age;
    }
 
    public void setAge(int age) {
        this.age = age;


}
```
people.properties内容如下：
```
age=10
name=xiaohua
history=beijing
```
##ConfigurationProperties
是类级别的注解
```
@PropertySource(value = {"classpath:people.properties"},ignoreResourceNotFound = false,encoding = "UTF-8",name = "people.properties")
@ConfigurationProperties(prefix = "female",ignoreUnknownFields=true,ignoreInvalidFields=true)
public class PeopleProperties {
 
    private String name;
 
    private int age;
     public String getName() {
        return name;
    }
 
    public void setName(String name) {
        this.name = name;
    }
 
    public int getAge() {
        return age;
    }
 
    public void setAge(int age) {
        this.age = age;

}
```
在PeopleProperties类中增加了ConfigurationProperties注解，并且指明了属性的前缀为female。这样Springboot在处理的时候，会去扫描当前类中的所有字段并进行属性的查找以及组装。比如我们配置的prefix = "female"，PeopleProperties类中有一个name字段，则female字段需要匹配的属性是prefix+字段=female.name。

@ConfigurationProperties(prefix = "female",ignoreUnknownFields=true,ignoreInvalidFields=true)
* ignoreUnknownFields：忽略未知的字段。

* ignoreInvalidFields：是否忽略验证失败的字段。这个怎么理解呢？比如我们在配置文件中配置了一个字符串类型的变量，类中的字段是int类型，那肯定会报错的。如果出现这种情况我们可以容忍，则需要配置该属性值为true。该参数值默认为false。
注意使用该注解，bean一定有Set与Get方法，否则取不出对应的属性值。
people.properties内容如下：
```
female.age=10
female.name=xiaohua
female.history=beijing
```
## @Value
通过@Value将外部配置文件的值动态注入到Bean中。配置文件主要有两类：

* application.properties。application.properties在spring boot启动时默认加载此文件
自定义属性文件。自定义属性文件通过@PropertySource加载。* @PropertySource可以同时加载多个文件，也可以加载单个文件。如果相同第一个属性文件和第二属性文件存在相同key，则最后一个属性文件里的key启作用。加载文件的路径也可以配置变量，如下文的${anotherfile.configinject}，此值定义在第一个属性文件config.properties
```
@Value("${app.name}")
    private String appName; // 这里的值来自application.properties，spring boot启动时默认加载此文件
```


# 无侵入容器
在设计上，Spring的IoC容器是一个高度可扩展的无侵入容器。所谓无侵入，是指应用程序的组件无需实现Spring的特定接口，或者说，组件根本不知道自己在Spring的容器中运行。这种无侵入的设计有以下好处：

* 应用程序组件既可以在Spring的IoC容器中运行，也可以自己编写代码自行组装配置；
* 测试的时候并不依赖Spring容器，可单独进行测试，大大提高了开发效率。

# 官方文档：
https://docs.spring.io/spring/docs/5.0.18.RELEASE/spring-framework-reference/core.html#spring-core
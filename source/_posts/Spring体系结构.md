title: Spring体系结构
author: wen
tags: []
categories:
  - spring
date: 2020-08-09 16:13:00
---
# 官网：
https://spring.io/

# Spring简介
Spring是一个轻量级Java开发框架，最早有Rod Johnson创建，目的是为了解决企业级应用开发的业务逻辑层和其他各层的耦合问题。它是一个分层的JavaSE/JavaEE full-stack（一站式）轻量级开源框架，为开发Java应用程序提供全面的基础架构支持。Spring负责基础架构，因此Java开发者可以专注于应用程序的开发。
## Spring优点：
Spring的优点:

* （1）方便解耦，简化开发

Spring就是一个大工厂，可以将所有对象创建和依赖的关系维护，交给Spring管理。

* （2）AOP编程的支持

Spring提供面向切面编程，可以方便的实现对程序进行权限拦截、运行监控等功能。

* （3）声明式事务的支持

只需要通过配置就可以完成对事务的管理，而无需手动编程。

* （4）方便程序的测试

Spring对Junit4支持，可以通过注解方便的测试Spring程序。

*（5）方便集成各种优秀框架

Spring不排斥各种优秀的开源框架，其内部提供了对各种优秀框架的直接支持（如：Struts、Hibernate、MyBatis等）。

*（6）降低JavaEE API的使用难度

Spring对JavaEE开发中非常难用的一些API（JDBC、JavaMail、远程调用等），都提供了封装，使这些API应用难度大大降低。

# spring 项目
在官网点击project 菜单 就见到spring很多项目
![spring project](Spring体系结构/1.png)


## springFramework
这是一个Spring的基础框架，提供了Spring 的核心功能，比如依赖注入、事务管理、面向方面编程等等
体系结构
![spring project](Spring体系结构/3.png)
Spring的核心容器是其他模块建立的基础，有Spring-core、Spring-beans、Spring-context、Spring-context-support和Spring-expression（String表达式语言）等模块组成。

* Spring-core模块：提供了框架的基本组成部分，包括控制反转（Inversion of Control，IOC）和依赖注入（Dependency Injection，DI）功能。
* Spring-beans模块：提供了BeanFactory，是工厂模式的一个经典实现，Spring将管理对象称为Bean。
* Spring-context模块：建立在Core和Beans模块的基础之上，提供一个框架式的对象访问方式，是访问定义和配置的任何对象的媒介。ApplicationContext接口是Context模块的焦点。
* Spring-context-support模块：支持整合第三方库到Spring应用程序上下文，特别是用于高速缓存（EhCache、JCache）和任务调度（CommonJ、Quartz）的支持。
* Spring-expression模块：提供了强大的表达式语言去支持运行时查询和操作对象图。这是对JSP2.1规范中规定的统一表达式语言（Unified EL）的扩展。该语言支持设置和获取属性值、属性分配、方法调用、访问数组、集合和索引器的内容、逻辑和算术运算、变量命名以及从Spring的IOC容器中以名称检索对象。它还支持列表投影、选择以及常用的列表聚合。
* Spring-aop模块：提供了一个符合AOP要求的面向切面的编程实现，允许定义方法拦截器和切入点，将代码按照功能进行分离，以便干净地解耦。
* Spring-aspects模块：提供了与AspectJ的集成功能，AspectJ是一个功能强大且成熟的AOP框架。
* Spring-jdbc模块：提供了一个JDBC的抽象层，消除了烦琐的JDBC编码和数据库厂商特有的错误代码解析。
*  Spring-orm模块：为流行的对象关系映射（Object-Relational Mapping）API提供集成层，包括JPA和Hibernate。使用Spring-orm模块可以将这些O/R映射框架与Spring提供的所有其他功能结合使用，例如声明式事务管理功能。
* Spring-oxm模块：提供了一个支持对象/XML映射的抽象层实现，例如JAXB、Castor、JiBX和XStream。
* Spring-jms模块（Java Messaging Service）：指Java消息传递服务，包含用于生产和使用消息的功能。自Spring4.1以后，提供了与Spring-messaging模块的集成。
* Spring-tx模块（事务模块）：支持用于实现特殊接口和所有POJO（普通Java对象）类的编程和声明式事务管理
* Spring-web模块：提供了基本的Web开发集成功能，例如多文件上传功能、使用Servlet监听器初始化一个IOC容器以及Web应用上下文。
* Spring-webmvc模块：也称为Web-Servlet模块，包含用于web应用程序的Spring MVC和REST Web Services实现。Spring MVC框架提供了领域模型代码和Web表单之间的清晰分离，并与Spring Framework的所有其他功能集成。
* Spring-websocket模块：Spring4.0以后新增的模块，它提供了WebSocket和SocketJS的实现。
* Portlet模块：类似于Servlet模块的功能，提供了Portlet环境下的MVC实现
* Spring-test模块支持使用JUnit或TestNG对Spring组件进行单元测试和集成测试。

## spring boot
我们原来运行Spring程序，需要在Maven或者Gradle中添加一些Spring的依赖，然后再通过这些构建工具提供的服务器来运行程序。使用Spring Boot，则可以免去这些繁复的工作。Spring Boot提供了一系列功能可以自动搜索、配置Spring程序。Spring Boot会将项目打包为一个可执行的jar文件，内部包含有tomcat这样的服务器，让我们可以直接以命令行的方式运行Spring程序。Spring Boot官方介绍说，它可以让你尽可能快的运行起Spring程序


## Springcloud
Spring Cloud是一系列框架的有序集合。它利用Spring Boot的开发便利性巧妙地简化了分布式系统基础设施的开发，如服务发现注册、配置中心、消息总线、负载均衡、断路器、数据监控等，都可以用Spring Boot的开发风格做到一键启动和部署。Spring Cloud并没有重复制造轮子，它只是将各家公司开发的比较成熟、经得起实际考验的服务框架组合起来，通过Spring Boot风格进行再封装屏蔽掉了复杂的配置和实现原理，最终给开发者留出了一套简单易懂、易部署和易维护的分布式系统开发工具包。
* Spring Cloud Netflix
　　是对Netflix开发的一套分布式服务框架的封装，包括服务的发现和注册，负载均衡、断路器、REST客户端、请求路由等。
* Spring Cloud Config
　　将配置信息中央化保存, 配置Spring Cloud Bus可以实现动态修改配置文件
* Spring Cloud Bus
　　分布式消息队列，是对Kafka, MQ的封装
* Spring Cloud Security
　　对Spring Security的封装，并能配合Netflix使用
* Spring Cloud Zookeeper
　　对Zookeeper的封装，使之能配置其它Spring Cloud的子项目使用
* Spring Cloud Eureka
Spring Cloud Eureka 是 Spring Cloud Netflix 微服务套件中的一部分，它基于Netflix Eureka 做了二次封装，主要负责完成微服务架构中的服务治理功能。
Spring Cloud对于中小型互联网公司来说是一种福音，因为这类公司往往没有实力或者没有足够的资金投入去开发自己的分布式系统基础设施，使用Spring Cloud一站式解决方案能在从容应对业务发展的同时大大减少开发成本。同时，随着近几年微服务架构和Docker容器概念的火爆，也会让Spring Cloud在未来越来越“云”化的软件开发风格中立有一席之地，尤其是在五花八门的分布式解决方案中提供了标准化的、全站式的技术方案，意义可能会堪比当年Servlet规范的诞生，有效推进服务端软件系统技术水平的进步。
## Spring Data
这是Spring关于处理数据的框架，其中包含了多个模块，可以让我们使用JPA操作数据、在Redis等非SQL数据库上存取数据等很多功能
## Spring Security
Spring的安全框架，支持在程序中设置安全权限，限制未授权的用户访问某些页面，也提供了一些加密功能方便地加密数据
## Spring Session
提供了一个Session的实现，帮助我们管理用户会话。
## Spring Integration
这个框架用来将Spring和其他框架、协议、服务集成起来，这些服务包括但不限于控制总线、FTP服务器、Web服务，社交服务、套接字、消息队列、邮件系统……Spring Integration提供了一些适配器，可以方便的和这些服务进行集成。
## Spring AMQP
用于开发AMQP的解决方案

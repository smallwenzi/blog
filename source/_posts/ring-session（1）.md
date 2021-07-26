title: spring session（1）
author: wen
tags:
  - spring
  - session
categories:
  - Spring
  - Spring boot
date: 2019-11-13 11:54:00
---
# 前言
在开始spring-session揭秘之前，先做下热脑（活动活动脑子）运动。主要从以下三个方面进行热脑：

* 为什么要spring-session
* 比较traditional-session方案和spring-session方案
* JSR340规范与spring-session的透明继承
# 一.为什么要spring-session
在传统单机web应用中，一般使用tomcat/jetty等web容器时，用户的session都是由容器管理。浏览器使用cookie中记录sessionId，容器根据sessionId判断用户是否存在会话session。这里的限制是，session存储在web容器中，被单台服务器容器管理。

但是网站主键演变，分布式应用和集群是趋势（提高性能）。此时用户的请求可能被负载分发至不同的服务器，此时传统的web容器管理用户会话session的方式即行不通。除非集群或者分布式web应用能够共享session，尽管tomcat等支持这样做。但是这样存在以下两点问题：

* 需要侵入web容器，提高问题的复杂
* web容器之间共享session，集群机器之间势必要交互耦合
基于这些，必须提供新的可靠的集群分布式/集群session的解决方案，突破traditional-session单机限制（即web容器session方式，下面简称traditional-session），spring-session应用而生。

# 二.比较traditional-session方案和spring-session方案

下图展示了traditional-session和spring-session的区别

![logo](ring-session（1）/1.png)
传统模式中，当request进入web容器，根据reqest获取session时，如果web容器中存在session则返回，如果不存在，web容器则创建一个session。然后返回response时，将sessonId作为response的head一并返回给客户端或者浏览器。

但是上节中说明了traditional-session的局限性在于：单机session。在此限制的相反面，即将session从web容器中抽出来，形成独立的模块，以便分布式应用或者集群都能共享，即能解决。

spring-session的核心思想在于此：将session从web容器中剥离，存储在独立的存储服务器中。目前支持多种形式的session存储器：Redis、Database、MogonDB等。session的管理责任委托给spring-session承担。当request进入web容器，根据request获取session时，由spring-session负责存存储器中获取session，如果存在则返回，如果不存在则创建并持久化至存储器中。

# JSR340规范与spring-session的透明继承

JSR340是Java Servlet 3.1的规范提案，其中定义了大量的api，包括：servlet、servletRequest/HttpServletRequest/HttpServletRequestWrapper、servletResponse/HttpServletResponse/HttpServletResponseWrapper、Filter、Session等，是标准的web容器需要遵循的规约，如tomcat/jetty/weblogic等等。

在日常的应用开发中，develpers也在频繁的使用servlet-api，比如：

以下的方式获取请求的session：
```
HttpServletRequest request = ...
HttpSession session = request.getSession(false);
```
其中HttpServletRequest和HttpSession都是servlet规范中定义的接口，web容器实现的标准。那如果引入spring-session，要如何获取session？

* 遵循servlet规范，同样方式获取session，对应用代码无侵入且对于developers透明化
* 全新实现一套session规范，定义一套新的api和session管理机制
两种方案都可以实现，但是显然第一种更友好，且具有兼容性。spring-session正是第一种方案的实现。

实现第一种方案的关键点在于做到透明和兼容

接口适配：仍然使用HttpServletRequest获取session，获取到的session仍然是HttpSession类型——适配器模式
类型包装增强：Session不能存储在web容器内，要外化存储——装饰模式
让人兴奋的是，以上的需求在Servlet规范中的扩展性都是予以支持！Servlet规范中定义一系列的接口都是支持扩展，同时提供Filter支撑扩展点。

# spring session入门
主要从以下两个方面来说spring-session：

* 特点
* 工作原理
## 一.特点
spring-session在无需绑定web容器的情况下提供对集群session的支持。并提供对以下情况的透明集成：

* HttpSession：容许替换web容器的HttpSession
* WebSocket：使用WebSocket通信时，提供Session的活跃
* WebSession：容许以应用中立的方式替换webflux的webSession

下面以项目接入spring session redis为例
# spring 项目
引入spring session的jar
代码：https://github.com/smallwenzi/testSpringSession
## 引入:
```
 <dependency>
	    	<groupId>org.springframework.session</groupId>
	    	<artifactId>spring-session</artifactId>
	    	<version>1.3.0.RELEASE</version>
		</dependency>
          <dependency>
	    	<groupId>org.springframework.session</groupId>
	    	<artifactId>spring-session</artifactId>
	    	<version>1.3.0.RELEASE</version>
		</dependency>
		
		<!-- Spring session redis 依赖start -->
		<dependency>
			<groupId>org.springframework.session</groupId>
			<artifactId>spring-session-data-redis</artifactId>
			<version>1.2.0.RELEASE</version>
		</dependency>
		<dependency>
		    <groupId>org.springframework.data</groupId>
		    <artifactId>spring-data-redis</artifactId>
		    <version>1.6.0.RELEASE</version>
		</dependency>
		<dependency>
		    <groupId>redis.clients</groupId>
		    <artifactId>jedis</artifactId>
		    <version>2.9.0</version>
		</dependency>
```
## 配置redis
applicationContext.xml
```
   <bean id="redisTemplate" class="org.springframework.data.redis.core.StringRedisTemplate">
       <property name="connectionFactory" ref="jedisConnectionFactory" />
   </bean>   
    <bean id="jedisConnectionFactory" class="org.springframework.data.redis.connection.jedis.JedisConnectionFactory">
        <property name="hostName" value="${redis.hostName}" />
        <property name="port" value="${redis.port}" />
        <property name="password" value="${redis.password}" />
        <property name="usePool" value="${redis.usePool}" />
        <property name="timeout" value="${redis.timeout}" />
    </bean>
```
redis.properties（自行更改redis 配置）
```
redis.hostName=localhost
redis.port=6379
redis.password=
redis.usePool=true
redis.timeout=10000

```

## 开启spring session
```
@Configuration
//redisNamespace区分每个项目spring session key唯一
@EnableRedisHttpSession(redisNamespace = "testSpringSession")
public class SpringSessionHttpConfig {

    private static final Logger logger = LoggerFactory.getLogger(SpringSessionHttpConfig.class);


    @Value("${redis.taskexecutor.corepoolsize}")
    private String corepoolsize;

    @Value("${redis.taskexecutor.maxpoolsize}")
    private String maxpoolsize;

    @Value("${redis.taskexecutor.keepaliveseconds}")
    private String keepaliveseconds;

    @Value("${redis.taskexecutor.queuecapacity}")
    private String queuecapacity;

    @Value("${redis.taskexecutor.threadnameprefix}")
    private String threadnameprefix;

//控制springsession 线程池 否则线程会无限制创建 导致oom 线程配置详细看看项目里test.properties
    @Bean
    public ThreadPoolTaskExecutor springSessionRedisTaskExecutor() {
        logger.info("JedisPool注入成功！！");
        ThreadPoolTaskExecutor springSessionRedisTaskExecutor = new ThreadPoolTaskExecutor();
        springSessionRedisTaskExecutor.setCorePoolSize(getRedisTaskexecutorStrToInt(this.corepoolsize, 16));
        springSessionRedisTaskExecutor.setMaxPoolSize(getRedisTaskexecutorStrToInt(this.maxpoolsize, 300));
        springSessionRedisTaskExecutor.setKeepAliveSeconds(getRedisTaskexecutorStrToInt(this.keepaliveseconds, 30));
        springSessionRedisTaskExecutor.setQueueCapacity(getRedisTaskexecutorStrToInt(this.queuecapacity, 500));
        springSessionRedisTaskExecutor.setThreadNamePrefix(this.threadnameprefix);
        return springSessionRedisTaskExecutor;
    }

    private int getRedisTaskexecutorStrToInt(String size, int defaultSize) {
        try {
            int sizeInt = Integer.parseInt(size);
            return sizeInt;
        } catch (Exception e) {
            return defaultSize;
        }
    }
```
## web.xml
```
  <filter>
  <!-- 必须这样命名 SpringHttpSessionConfiguration类中定义了springSessionRepositoryFilter类 -->
	<filter-name>springSessionRepositoryFilter</filter-name>
	<filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>	
  </filter>
  <filter-mapping>
	<filter-name>springSessionRepositoryFilter</filter-name>
    <!-- 需要spring session的访问路径 -->
	<url-pattern>/*</url-pattern> 
  </filter-mapping>

```
本地分别部署tomcat 进行测试http://localhost:8082/testSpringSession/testSession.do
http://localhost:8080/testSpringSession/testSession.do
输出的session id是否一致
若一致则spring session成功


# spring boot项目
项目：https://github.com/smallwenzi/testSpringBootSession
## 引入jar
```
	           <dependency>
	    <groupId>org.springframework.session</groupId>
	    <artifactId>spring-session</artifactId>
	   </dependency>
	   	<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-redis</artifactId>
		</dependency>
	 </dependencies>
```
## 开启springsession
```
//redisNamespace区分每个项目spring session key唯一
@EnableRedisHttpSession(redisNamespace = "testSpringBootSession")
public class SpringSessionHttpConfig {
    private static final Logger logger = LoggerFactory.getLogger(SpringSessionHttpConfig.class);
    @Value("${redis.taskexecutor.corepoolsize}")
    private String corepoolsize;

    @Value("${redis.taskexecutor.maxpoolsize}")
    private String maxpoolsize;

    @Value("${redis.taskexecutor.keepaliveseconds}")
    private String keepaliveseconds;

    @Value("${redis.taskexecutor.queuecapacity}")
    private String queuecapacity;

    @Value("${redis.taskexecutor.threadnameprefix}")
    private String threadnameprefix;

//控制springsession 线程池 否则线程会无限制创建 导致oom 线程配置详细看看项目里application.properties
    @Bean
    public ThreadPoolTaskExecutor springSessionRedisTaskExecutor() {
        logger.info("JedisPool注入成功！！");
        ThreadPoolTaskExecutor springSessionRedisTaskExecutor = new ThreadPoolTaskExecutor();
        springSessionRedisTaskExecutor.setCorePoolSize(getRedisTaskexecutorStrToInt(this.corepoolsize, 16));
        springSessionRedisTaskExecutor.setMaxPoolSize(getRedisTaskexecutorStrToInt(this.maxpoolsize, 300));
        springSessionRedisTaskExecutor.setKeepAliveSeconds(getRedisTaskexecutorStrToInt(this.keepaliveseconds, 30));
        springSessionRedisTaskExecutor.setQueueCapacity(getRedisTaskexecutorStrToInt(this.queuecapacity, 500));
        springSessionRedisTaskExecutor.setThreadNamePrefix(this.threadnameprefix);
        return springSessionRedisTaskExecutor;
    }

    private int getRedisTaskexecutorStrToInt(String size, int defaultSize) {
        try {
            int sizeInt = Integer.parseInt(size);
            return sizeInt;
        } catch (Exception e) {
            return defaultSize;
        }
    }
}

```
## application.properties
```
#本地测试设置随机端口 （自行更改redis 配置）
server.port=0
# redis start
#spring.redis.database=0
spring.redis.password=
spring.redis.pool.max-idle=10
spring.redis.pool.min-idle=0
spring.redis.pool.max-active=100
spring.redis.pool.max-wait=1000
spring.redis.database=0
# Redis\u670D\u52A1\u5668\u5730\u5740
spring.redis.host=localhost
# Redis\u670D\u52A1\u5668\u8FDE\u63A5\u7AEF\u53E3
spring.redis.port=6379

# redis end
#spring session\u4f7f\u7528\u5b58\u50a8\u7c7b\u578b
spring.session.store-type=redis

##spring-session redis \u7ebf\u7a0b\u6c60\u914d\u7f6e
redis.taskexecutor.corepoolsize=16
redis.taskexecutor.maxpoolsize=300
redis.taskexecutor.keepaliveseconds=30
redis.taskexecutor.queuecapacity=500
redis.taskexecutor.threadnameprefix=testSpringBootSession-spring-session-redis-executor-thread:

```
本地分别启动 WebApplication进行测试http://localhost:port/testSession
http://localhost:port/
testSession
输出的session id是否一致
若一致则spring session成功

有兴趣可以读[《spring sesson源码》](https://smallwenzi.github.io/2019/11/21/SpringSession%E6%BA%90%E7%A0%81/ "《spring sesson源码》")
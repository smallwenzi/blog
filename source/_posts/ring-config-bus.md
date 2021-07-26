title: spring config bus
author: wen
tags:
  - spring config
  - spring cloud
categories:
  - spring cloud
date: 2019-05-29 17:06:00
---
config pom.xml
```
 <dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-starter-bus-amqp</artifactId>
			</dependency>
        <dependency>
```
/configuration/src/main/resources/application-native.properties
```
spring.cloud.bus.enabled=true
spring.cloud.bus.trace.enabled=true
spring.rabbitmq.addresses=${RABBITMQ_ADDRESS}
spring.rabbitmq.username=${RABBITMQ_USERNAME}
spring.rabbitmq.password=${RABBITMQ_PASSWORD}
spring.rabbitmq.virtual-host=${RABBITMQ_VIRTUAL_HOST}
## \u5237\u65B0\u65F6\uFF0C\u5173\u95ED\u5B89\u5168\u9A8C\u8BC1
management.security.enabled=false
management.endpoints.web.exposure.include=bus-refresh
```

刷新链接：
http://ip:6081/actuator/bus-refresh


客户端
pom.xml
```
<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-starter-bus-amqp</artifactId></dependency>
```
HelloController
```
@RestController
@RefreshScope
// 使用该注解的类，会在接到SpringCloud配置中心配置刷新的时候，自动将新的配置更新到该类对应的字段中。
class HelloController {

    @Value("${business.wx.appid}")
    private String hello;

    @RequestMapping("/hello")
    public String from() {
        return this.hello;
    }
}
```

配置
```
######## bus-amqp
## 刷新时，关闭安全验证
management.security.enabled=false
management.endpoints.web.exposure.include=bus-refresh
## 开启消息跟踪
spring.cloud.bus.enabled=true
spring.cloud.bus.trace.enabled=true
#mq的地址
spring.rabbitmq.addresses=
#mq的用户名
spring.rabbitmq.username=
#mq的密码
spring.rabbitmq.password=
spring.rabbitmq.publisherConfirms=false
spring.rabbitmq.publisherReturns=false
spring.rabbitmq.virtual-host=/

```

测试连接
测试入口：http://ip:port/hello
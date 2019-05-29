title: spring admin配置
author: wen
tags:
  - springAdmin
  - spring cloud
categories:
  - spring cloud
date: 2019-05-29 16:54:00
---
spring admin 接入 eureka

pom.xml
```
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.amway.springcloud</groupId>
  <artifactId>springamdinserver</artifactId>
  <version>2.1</version>

   <name>基础设施:springamdinserver</name>

  <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.0.RELEASE</version>
    </parent>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>
  </properties>

  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <scope>test</scope>
    </dependency>
    <dependency>
    <groupId>de.codecentric</groupId>
    <artifactId>spring-boot-admin-starter-server</artifactId>
    <version>2.1.4</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
  </dependencies>

   <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>Finchley.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
  <build>
   
    <plugins>
     <!-- 源码格式 -->
            <plugin>  
                <groupId>com.googlecode.maven-java-formatter-plugin</groupId>  
                <artifactId>maven-java-formatter-plugin</artifactId>  
                <version>0.4</version>  
                <configuration>  
                    <configFile>${project.basedir}/code-style.xml</configFile>  
                </configuration>  
                <executions>  
                    <!--                 	<execution>   -->
                    <!--                     	<goals>   -->
                    <!--                         	<goal>format</goal>   -->
                    <!--                         </goals>   -->
                    <!--                     </execution>   -->
                </executions>  
            </plugin>
            <!-- 单元测试与覆盖率 -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>2.22.1</version><!--$NO-MVN-MAN-VER$-->
                <dependencies>
                    <dependency>
                        <groupId>org.apache.maven.surefire</groupId>
                        <artifactId>surefire-junit47</artifactId>
                        <version>2.21.0</version>
                    </dependency>
                </dependencies>
                <configuration>
                    <skip>true</skip>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-report-plugin</artifactId>
                <version>2.21.0</version>
                <configuration>
                    <includes>
                        <include>**/*Test.java</include>
                    </includes>
                </configuration>
            </plugin>
            <plugin>  
                <groupId>org.codehaus.mojo</groupId>  
                <artifactId>cobertura-maven-plugin</artifactId>  
                <version>2.7</version> 
                <configuration>
                    <formats>
                        <format>html</format>
                        <format>xml</format>
                    </formats>
                </configuration> 
            </plugin>
            <!--编译-->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.0</version><!--$NO-MVN-MAN-VER$-->
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                    <encoding>UTF-8</encoding>
                </configuration>
            </plugin>
            <!-- javadoc文档 -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-javadoc-plugin</artifactId>
                <version>3.0.1</version><!--$NO-MVN-MAN-VER$-->
                <configuration>
                    <aggregate>true</aggregate>
                </configuration>
            </plugin>
            <!-- spring boot -->
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <executions>
                    <execution>
                        <goals>
                            <goal>repackage</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
            <plugin>
                <artifactId>maven-antrun-plugin</artifactId>
                <version>1.8</version><!--$NO-MVN-MAN-VER$-->
                <executions>
                    <execution>
                        <id>copy-lib-src-webapps</id>
                        <phase>package</phase>
                        <configuration>
                            <target>
                                <copy todir="${project.basedir}/docker">
                                    <fileset dir="${project.basedir}/target">
                                        <include name="*.jar" />
                                    </fileset>
                                </copy>
                            </target>
                        </configuration>
                        <goals>
                            <goal>run</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
            <plugin>
                <groupId>com.spotify</groupId>
                <artifactId>docker-maven-plugin</artifactId>
                <version>1.2.0</version>
                <configuration>
                    <!-- imageName必须全为小写 -->
                    <imageName>springboot-unit-test:1.0</imageName>
                    <dockerDirectory>${project.basedir}/docker</dockerDirectory>
                    <resources>
                        <resource>
                            <targetPath>/</targetPath>
                            <directory>${project.build.directory}</directory>
                            <include>${project.build.finalName}.jar</include>
                        </resource>
                    </resources>
                </configuration>
            </plugin>
      </plugins>
  </build>
</project>


```

application.properties

```
spring.application.name=springadminserver
server.port=8080
#using the info endpoint
info.tags.environment=test
#配置eureka
eureka.client.serviceUrl.defaultZone=${EUREKA_SERVICEURL}
eureka.instance.prefer-ip-address=true
spring.cloud.inetutils.preferredNetworks=${PREFERRED_IP_PATTERN:.*}
eureka.instance.preferIpAddress=true
eureka.instance.instance-id=${spring.cloud.client.ip-address}:${server.port}
eureka.instance.leaseRenewalIntervalInSeconds=10
eureka.instance.health-check-url-path=/actuator/health
eureka.client.registryFetchIntervalSeconds=5
#公开的站点
management.endpoints.web.exposure.include=*
management.endpoints.health.show-details=ALWAYS
# 安全密码
spring.security.user.name=user
spring.security.user.password=123456
```

SpringBootAdminApplication
```
package com.springamdinserver;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.web.authentication.SavedRequestAwareAuthenticationSuccessHandler;

import de.codecentric.boot.admin.server.config.AdminServerProperties;
import de.codecentric.boot.admin.server.config.EnableAdminServer;

@Configuration
@EnableAutoConfiguration
@EnableAdminServer
@EnableDiscoveryClient
public class SpringBootAdminApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringBootAdminApplication.class, args);
    }

    @Configuration
    public static class SecurityPermitAllConfig extends WebSecurityConfigurerAdapter {
        private final String adminContextPath;

        public SecurityPermitAllConfig(AdminServerProperties adminServerProperties) {
            this.adminContextPath = adminServerProperties.getContextPath();
        }

        @Override
        protected void configure(HttpSecurity http) throws Exception {
            // @formatter:off
            SavedRequestAwareAuthenticationSuccessHandler successHandler =
                    new SavedRequestAwareAuthenticationSuccessHandler();
            successHandler.setTargetUrlParameter("redirectTo");

            http.authorizeRequests().antMatchers(adminContextPath + "/assets/**").permitAll()
                    .antMatchers(adminContextPath + "/login").permitAll().anyRequest().authenticated().and()
                    .formLogin().loginPage(adminContextPath + "/login").successHandler(successHandler).and().logout()
                    .logoutUrl(adminContextPath + "/logout").and().httpBasic().and().csrf().disable();
            // @formatter:on
        }
    }
}

```
客户端配置
```
management.endpoints.web.exposure.include=*
management.endpoints.health.show-details=ALWAYS
##日志路径
logging.path=/applog/logincenterWxBindingService
```
重启服务

控制台：
http://localhost:8080/
![logo](pring-admin配置/1.png)
eurka上服务
![logo](pring-admin配置/2.png)
每个服务 内存情况
![logo](pring-admin配置/3.png)
查看日志
![logo](pring-admin配置/4.png)
流量情况：
![logo](pring-admin配置/5.png)


参考：https://github.com/codecentric/spring-boot-admin
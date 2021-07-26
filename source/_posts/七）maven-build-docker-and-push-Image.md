title: 七）maven build docker and push Image
author: wen
tags:
  - docker
  - maven
categories:
  - docker
date: 2018-05-08 17:36:00
---
链接：[docker安装本地仓库](/2018/05/08/（六）docker安装本地仓库（root用户）/)
使用maven 打包jar成docker镜像 并发布到镜像库
# 1 maven settings.xml配置server 节点添加
（这是私有库 用户名密码）
```
	<server>
			<id>docker-hub</id>
			<username>testuser</username>
			<password>testpassword</password>
       </server>
```
# 2 项目 pom.xml
properties 节点

```
      <docker.image.prefix>reg.itmuch.com</docker.image.prefix>
     <docker.registry>https://reg.itmuch.com/v1</docker.registry>
        <tag.version>1.0.0</tag.version>
         <docker.servcerId>docker-hub</docker.servcerId>
```
maven 配置组件
```
         <!-- docker的maven插件，官网：https://github.com/spotify/docker-maven-plugin -->
            <plugin>
                <groupId>com.spotify</groupId>
                <artifactId>docker-maven-plugin</artifactId>
                <version>1.0.0</version>
                <dependencies>
                <dependency>
					    <groupId>commons-logging</groupId>
					    <artifactId>commons-logging</artifactId>
					    <version>1.1.1</version>
					</dependency>
                </dependencies>
                <configuration>
                   	<!-- 镜像名:tag  -->
                   <imageName>${docker.image.prefix}/${project.artifactId}:${tag.version}</imageName>
				    <!-- 指定Dockerfile所在的路径  -->
                    <dockerDirectory>${project.basedir}/docker</dockerDirectory>
                    <!-- 以下两行是为了docker push到DockerHub使用的。 -->
                    <serverId>${docker.servcerId}</serverId>
                    <registryUrl>${docker.registry}</registryUrl>
        			   <resources>
                        <resource>
                            <targetPath>/</targetPath>
                            <directory>${project.build.directory}</directory>
                            <include>${project.build.finalName}.jar</include>
                        </resource>
                    </resources>
                </configuration>
            </plugin>
```

参考https://github.com/spotify/docker-maven-plugin

# 3 dockerFile文件放在项目根目录下docker目录
```
FROM openjre
MAINTAINER killko <ydhana@isoftstone.com>
ENV EUREKA_INSTANCE_NAME=eureka
ENV EUREKA_SERVICEURL=http://eureka:6080/eureka
EXPOSE 6080
#统一时区
RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai  /etc/localtime

COPY eureka*.jar /Eureka.jar

ENTRYPOINT ["java","-jar","-Djava.net.preferIPv4Stack=true","/Eureka.jar"]

```
# 4 maven build docker and push Image：
mvn clean package docker:build -DpushImage

（需要注意：运行环境必须有docker）
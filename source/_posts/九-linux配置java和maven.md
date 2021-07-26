title: 九 linux配置java和maven
author: wen
tags:
  - java
  - maven
categories:
  - linux
date: 2018-05-11 22:46:00
---
安装链接：
[oracle VM安装red hat liunx](/2018/05/08/）oracle-VM安装red-hat-liunx/)

# 安装java
1 从http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html 页面下载jdk-8u171-linux-x64.tar.gz
2 上传文件至linux服务器 /app
3 在/app目录 执行
```
tar -xvf jdk-8u171-linux-x64.tar.gz
```

# 安装maven:
1 在linux里/app执行：wget https://archive.apache.org/dist/maven/maven-3/3.2.3/binaries/apache-maven-3.2.3-bin.tar.gz 
2 在/app目录 执行
```
tar -xvf  apache-maven-3.2.3-bin.tar.gz
```
3 在/app/apache-maven-3.2.3/bin 目录执行
```
chmod +x ./mvn
```

# 配置环境(root用户)
```
vi /etc/profile
```
配置maven 和java环境
```
export MAVEN_HOME=/app/apache-maven-3.2.3
export MAVEN_HOME
export PATH=$PATH:$MAVEN_HOME/bin
export JAVA_HOME=/app/jdk1.8.0_171
export JAVA_HOME
export PATH=$PATH:$JAVA_HOME/bin
```
保存完成
```
source /etc/profile
```

# 验证
```
java -version
```

```
mvn -v
```
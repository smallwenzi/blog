title: 八）安装jenkins
author: wen
tags:
  - jenkins
categories: []
date: 2018-05-08 18:03:00
---
参考：

1[oracle VM安装red hat liunx](/2018/05/08/）oracle-VM安装red-hat-liunx/)

2[升级RPM和YUM 包](/2018/05/08/）Red-Hat-升级-rpm软件、YUM软件（root用户）/)


# 1 安装java环境
```
yum install -y java
```

# 2 安装wget
```
yum install -y wget
```

# 3 获取jenkins的配置
```
 wget -O /etc/yum.repos.d/jenkins.repo http://pkg.jenkins.io/redhat/jenkins.repo
```
# 4 导入jenkinskey
```
 rpm --import http://pkg.jenkins.io/redhat/jenkins.io.key
 ``

# 5 安装jenkins
```
yum install jenkins
```
# 6 jenkins的配置文件： 
```
/etc/sysconfig/jenkins
```
# 7 运行 jenkins
```
service jenkins start
```
# 8 查看日志：
```
tail-f  /var/log/jenkins/jenkins.log
```
![logo](安装jenkins/11278290-b7f1229f3c16a906.png)
211是jenkins key

如果不能访问请关闭防火墙
iptables -F

# 9 浏览器访问：192.168.137.8：8080
![logo](安装jenkins/11278290-e2f6e1054f7f923a.png)
输入21120240a2cb4e8e83065dc7c71a5b88  
点击确定

# 10 安装推荐的插件
![logo](安装jenkins/11278290-9e1548cc032d002c.png)
# 11  等等安装完成
![logo](安装jenkins/11278290-a2000d90816960d4.png)
12 创建用户:  输入完信息 点击 确定：  
![logo](安装jenkins/11278290-364c455c39845e06.png)
13 点击save and finish

![logo](安装jenkins/11278290-6cce522c3a59044b.png)
14 安装成功
![logo](安装jenkins/11278290-aab55082195f5c56.png)

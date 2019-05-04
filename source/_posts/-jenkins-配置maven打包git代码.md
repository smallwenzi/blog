title: 十二 jenkins 配置maven打包git代码
author: wen
tags:
  - git
  - maven
  - docker
  - jenkins
categories:
  - jenkins
date: 2018-05-11 23:36:00
---
一 [jenkins-配置git证书](/2018/05/11/-jenkins-配置git证书/)
二 [安装jenkins](/2018/05/08/安装jenkins/)
三 [jenkins 配置java和Maven环境](/2018/05/11/enkinx-配置任务/)

![logo](-jenkins-配置maven打包git代码/总图.png)
# 一配置任务
1.1点击 创建一个新任务
![logo](-jenkins-配置maven打包git代码/1.png)
1.2 输入名称  选择自由风格的软件项目
![logo](-jenkins-配置maven打包git代码/2.png)
1.3新增文本参数
1.3.1参数docker.image.prefix: 私有库域名
![logo](-jenkins-配置maven打包git代码/3.png)
1.3.2参数docker.registry：私有库地址
![logo](-jenkins-配置maven打包git代码/4.png)
1.3.3参数docker.servcerId：刚才maven配置 私有库用户名和密码
![logo](-jenkins-配置maven打包git代码/5.png)
1.3.4参数version 
![logo](-jenkins-配置maven打包git代码/6.png)
1.4配置 git库
![logo](-jenkins-配置maven打包git代码/7.png)
1.5构建步骤添加 执行Shell脚本
![logo](-jenkins-配置maven打包git代码/8.png)
```
脚本命令：
echo $WORKSPACE
echo $MAVEN_HOME
cd $WORKSPACE/Eureka
$MAVEN_HOME/bin/mvn clean package docker:build -DpushImage -Ddocker.image.prefix=$docker_image_prefix -Ddocker.registry=$docker_registry -Ddocker.servcerId=$docker_servcerId -Dtag.version=$version
```
保存

# 二 配置jenkins用户可以执行docker 命令
liunx用户 使用jenkins 用户启动jenkins
```
vi /etc/systemd/system/multi-user.target.wants/docker.service
 ExecStart=/usr/bin/dockerd  -G jenkins

```
 -G表示 jenkins用户可以执行docker 命令
 ![logo](-jenkins-配置maven打包git代码/9.png)
 
 重新加载配置
 ```
  systemctl daemon-reload
  ```
  重新启动docker
  ```
 systemctl restart docker
 ```

# 三执行任务 
3.1 点击 Build with Parameters
![logo](-jenkins-配置maven打包git代码/10.png)
3.2 点击开始创建
![logo](-jenkins-配置maven打包git代码/11.png)
3.3查看脚本执行情况
![logo](-jenkins-配置maven打包git代码/12.png)
3.4成功后查看私有仓库目录
![logo](-jenkins-配置maven打包git代码/132.png)
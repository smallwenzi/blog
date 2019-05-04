title: 十）jenkins 配置java和Maven环境
author: wen
tags:
  - docker
  - jenkins
  - java
  - maven
categories:
  - jenkins
date: 2018-05-11 23:01:00
---
参考：
[linux配置java和maven](/2018/05/11/九-linux配置java和maven/)
[docker安装本地仓库](2018/05/08/（六）docker安装本地仓库（root用户）/)
[安装jenkins](/2018/05/08/安装jenkins/)

# 一配置maven
/app/apache-maven-3.2.3/conf/settings.xml
库位置
![logo](enkinx-配置任务/1.png)
以下是私有docker库 用户名密码）
![logo](enkinx-配置任务/2.png)

# 二 jenkins 配置java和Maven环境
2.1点击系统管理
![logo](enkinx-配置任务/11278290-aab55082195f5c56.png)
2.2 点击 全局工具配置 
![logo](enkinx-配置任务/3.png)
2.3 如下图配置 java和maven.
![logo](enkinx-配置任务/4.png)

![logo](enkinx-配置任务/5.png)
2.4 点击 保存 再次点击 系统设置
![logo](enkinx-配置任务/6.png)
2.5 再次配置java和maven目录
![logo](enkinx-配置任务/7.png)
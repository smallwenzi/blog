title: 十一 jenkins 配置git证书
author: wen
tags:
  - jenkins
  - git
categories:
  - jenkins
date: 2018-05-11 23:26:00
---
[linux 安装 GIT（root用户）](/2018/05/08/三）linux-安装-GIT（root用户）/)
[安装jenkins](/2018/05/08/安装jenkins/)

# 一 linux 生成RSA key
linux远程登录ssh ywtan@localhost
```
  ssh-keygen -t rsa
```
![logo](-jenkins-配置git证书/1.png)

全部回车
```
cat .ssh/id_rsa(这是刚才设置 rsa_key)
```

# 二 jenkins 配置git证书
2.1 在jenkins 页面点击credentials
![logo](-jenkins-配置git证书/2.png)
2.2 点击 system
![logo](-jenkins-配置git证书/3.png)
2.3 点击 Global credentials (unrestricted)
![logo](-jenkins-配置git证书/4.png)
2.4 add Credentials
![logo](-jenkins-配置git证书/5.png)
2.5 如图所示填写， key是刚才生成的key，最后点击保存
![logo](-jenkins-配置git证书/6.png)
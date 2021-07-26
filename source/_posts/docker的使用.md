title: docker的使用
author: wen
tags:
  - docker
categories:
  - docker
date: 2018-06-11 20:50:00
---


1 docker build出现Temporary failure resolving
```
vi/etc/systemd/system/multi-user.target.wants/docker.service ExecStart=/usr/bin/dockerd 
添加 --dns=8.8.8.8
重新加载配置
>systemctl daemon-reload 
 --启动docker
>systemctl start docker
```
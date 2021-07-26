title: （五）更新docker 版本（root用户）
author: wen
tags:
  - docker
categories: []
date: 2018-05-08 17:00:00
---
参考：[安装docker 操作](/2018/05/08/（四）安装docker（root用户）/)

# 1 删除旧版本（以下操作都是使用root用户）
```
yum remove docker \

docker-client \

docker-client-latest\ 

docker-common \

docker-latest \

docker-latest-logrotate \

docker-logrotate \

docker-selinux \

docker-engine-selinux \

docker-engine

```

# 2 确保docker-ce-edge最新版本
```
yum-config-manager--enable docker-ce-edge
```
# 3 安装docker
```
 yum install docker-ce
```

# 4 设置集群功能
```
vi /etc/systemd/system/multi-user.target.wants/docker.service 
在
ExecStart=/usr/bin/dockerd 添加 --experimental=true  表示启用集群功能
```
# 5 重新加载配置
```
systemctl daemon-reload
```

# 6 启动服务
```
sysctemctl start docker
```

# 7 查看版本：
```
docker version
```
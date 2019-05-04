title: （四）安装docker（root用户）
author: wen
tags:
  - docker
categories: []
date: 2018-05-08 15:59:00
---
链接：
[oracle VM安装red hat liunx](/2018/05/08/）oracle-VM安装red-hat-liunx/)

[升级RPM和YUM 包](/2018/05/08/）Red-Hat-升级-rpm软件、YUM软件（root用户）/)

# 1 安装docker（以下操作都是使用root用户）
```
--安装驱动（red-hat为例,不同操作系统，驱动不同详细看官网教程）
> yum install -y yum-utils device-mapper-persistent-data lvm2  

--添加docker仓库
> yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

--清除缓存
> yum clean all

--重新更新缓存
> yum makecache

--确保docker engine最新
> yum-config-manager --enable docker-ce-edge

--安装docker
> yum install docker-ce

开机启动
> systemctl enable docker

开启集群权限
>vi /etc/systemd/system/multi-user.target.wants/docker.service ExecStart=/usr/bin/dockerd 
添加
--experimental=true 

重新加载配置
>systemctl daemon-reload 

 --启动docker
>systemctl start docker
```

参考：[docker官网](https://docs.docker.com/install/linux/docker-ce/centos/)

docker 常用命令 参考[docker 命令官网](https://docs.docker.com/engine/reference/run/)

# 2 查看docker 版本
```
docker version --目前最新版本是 17.12.1-ce
```
# 3 查看docker 信息
```
docker info
```
# 4 运行镜像
```
docker run hello-word --本地没有会找镜像库 
```
关于run 一些参数详细参考[docker官网](https://docs.docker.com/engine/reference/run/)



# 5 下载镜像
```
docker pull alpine
```
# 6 制作java环境镜像：
使用DockerFile 进行编写 ,参考[dockerFile官网](https://docs.docker.com/engine/reference/builder/#usage)

```
FROM alpine
MAINTAINER killko <killko@qq.com>
ENV TIME_ZONE Asiz/Shanghai
RUN echo '@edge http://nl.alpinelinux.org/alpine/edge/main' >> /etc/apk/repositories && \
    echo '@community http://nl.alpinelinux.org/alpine/edge/community' >> /etc/apk/repositories && \
    apk update && \
    apk upgrade && \
    apk add ca-certificates openjdk8-jre-base@community && \
    apk del ca-certificates && \
    apk add --no-cache tzdata && \
    echo "${TIME_ZONE}" > /etc/timezone && \
    ln -sf /usr/share/zoneinfo/${TIME_ZONE} /etc/localtime && \
    rm -rf /tmp/* /var/cache/apk/*
RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai  /etc/localtime
```
编写完成进行build
```
>docker build  -t openjre:latest . --表示在当前目录进行制作镜像
>docker images --查看镜像
>docker run -it openjre sh --进行查看java是否安装成功
```


# 7 导入镜像
```
docker load < openjre.tar
```

# 8 导出镜像：
```
docker save openjre:latest  openjre.tar
```

# 9 查看镜像
```
docker images
```

# 10 查看正在运行容器
```
docker ps -a --查看容器id 运行状态 映射端口
```

# 11 查看容器日志
```
docker logs -f 容器id --容器id从docker ps -a获取
```

# 12 停止容器
```
docker stop 容器id ----容器id从docker ps -a获取
```

# 13 删除容器
```
docker rm 容器id --容器id从docker ps -a获取
```
# 14 删除镜像
```
docker rmi imagename:tag --imagename:tag imagename表示镜像名称 tag表示版本 这些信息可以从docker images进行获取
```

# 15 进入容器 运行sh命令：
docker exec -it 容器id sh --容器id从docker ps -a获取

# 16 添加普通用户操作docker
```
vi etc/systemd/system/multi-user.target.wants/docker.service （操作服务）

ExecStart=/usr/bin/dockerd -G jenkins  #表示jenkins用户组可以操作docker命令

保存后

systemctl daemon-reload --重新加载服务

systemctl restart docker --重启服务
```
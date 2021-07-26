title: （六）docker安装本地仓库（root用户）
author: wen
tags:
  - docker
categories: []
date: 2018-05-08 17:09:00
---
参考：
[安装docker](/2018/05/08/（四）安装docker（root用户）/)

创建使用仓库，并设置https 及安全访问

# 1下载镜像 
```
 > docker pull registry 
```
# 2 创建目录
```
> mkdir -p ~/certs 
```

# 3 进入目录
```
> cd ~/certs  
```

# 4 创建域名证书
```
> openssl genrsa -out rep.itmuch.com.key 2048 
```

# 5 创建证书
```
> openssl req -newkey rsa:4096 -nodes -sha256 -keyout \ rep.itmuch.com.key -x509 -days 365 -out \ rep.itmuch.com.crt  

Country Name (2 letter code) [XX]:       # 你的国家名称

CN State or Province Name (full name) []:GD    # 省份

Locality Name (eg, city) [Default City]:GZ      # 所在城市

Organization Name (eg, company) [DefaultCompany Ltd]:test     # 组织名称

Organizational Unit Name (eg, section)[]:test      # 组织单元名称

Common Name (eg, your name or your server'shostname) []:rep.itmuch.com    # 域名

Email Address []:     # 邮箱 
```
# 3 复制证书至docker
## 3.1创建目录
```
> mkdir -p /etc/docker/certs.d/rep.itmuch.com
```
## 3.2 复制证书至docker目录
```
> cp ~/certs/rep.itmuch.com.crt /etc/docker/certs.d/rep.itmuch.com/ 
```
## 3.3 重启docker
```
> systemctl restart docker  
```

# 4 设置用户名和密码
## 4.1 安装 httpd-tools
```
> yum install httpd-tools 
```
## 4.2 当前登录用户
```
> cd~  
```
## 4.3 创建目录
```
>mkdir auth 
```
## 4.4创建用户名和密码 
```
htpasswd -Bbn testuser testpassword > auth/htpasswd  
```
## 5 运行
```
docker run -d -p 443:5000   \-v /root/certs:/certs \ -v/root/auth:/auth \-v /opt/docker-image:/var/lib/registry \-e STORAGE_PATH=/opt/docker-image \-e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/rep.itmuch.com.crt\-e REGISTRY_HTTP_TLS_KEY=/certs/rep.itmuch.com.key\ -e"REGISTRY_AUTH=htpasswd" \  -e"REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" \  -eREGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd \registry
```
# host主机名
```
 echo '192.168.137.8 rep.itmuch.com'>> /etc/hosts 
```


# 登录私有库
```
docker login rep.itmuch.com
```
push或pull需要登录否则不能报错

# push到私有库
```
docker tag alpine rep.itmuch.com/alpine

docker push rep.itmuch.com/alpine
```

# 查看镜像目录
```
https://rep.itmuch.com/v2/_catalog
```

# 查看镜像 版本列表
```
https://rep.itmuch.com/v2/alpine/tags/list
```
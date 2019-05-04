title: docker -mysql
author: wen
tags:
  - docker
  - mysql
categories: []
date: 2018-05-08 17:28:00
---

参考：
[安装docker](/2018/05/08/（四）安装docker（root用户）/)

# 下载镜像
```
docker pull mysql
```
# 运行镜像
```
docker run --name mysql -p 3306:3306 -v /app/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=root -d mysql
```

# 进入mysql进行设置
```
docker ps -a  --获取mysql 容器的id

docker exec -it 容器id sh
```

# 进入Mysql后登录mysql
```
mysql -uroot -proot

若报root@localhost,请试试mysql -uroot
```
# 创建数据库用户
```
create USER springcloud IDENTIFIED WITH mysql_native_password BY '123456';

grant all privileges on *.* to springcloud;

create database springclouddb
```

# 报错信息
若show databases 报错 
若是 缺没有此mysql.infoschema用户
则执行以下sql
```
create USER 'mysql.infoschema' IDENTIFIED WITH mysql_native_password BY '123456';

create USER 'mysql.session' IDENTIFIED WITH mysql_native_password BY '123456';

create USER 'mysql.sys'  IDENTIFIED WITH mysql_native_password BY '123456';

create USER springcloud IDENTIFIED WITH mysql_native_password BY '123456';

grant all privileges on *.* to 'mysql.infoschema';

grant all privileges on *.* to 'mysql.session';

grant all privileges on *.* to 'mysql.sys';
```
title: （二）Red Hat 升级 rpm YUM软件
author: wen
tags:
  - rpm
  - yum
categories:
  - linux
date: 2018-05-08 15:32:00
---
以下操作是root用户

1 [oracle VM安装red hat liunx](/2018/05/08/）oracle-VM安装red-hat-liunx/)

rpm 下载链接：http://mirrors.163.com/centos/7/os/x86_64/Packages/

# 查看版本
```
> rpm version
```
![logo](）Red-Hat-升级-rpm软件、YUM软件（root用户）/11278290-76fc51b36f51535f.png)

# 升级rpm
```
>rpm -Uvh rpm-4.11.3-25.el7.x86_64.rpm  rpm-build-libs-4.11.3-25.el7.x86_64.rpm rpm-libs-4.11.3-25.el7.x86_64.rpm rpm-python-4.11.3-25.el7.x86_64.rpm --force–i  
```
![logo](）Red-Hat-升级-rpm软件、YUM软件（root用户）/11278290-2b0c840c69622e92.png)
# 删除旧的yum
```
> rpm -e yum-metadata-parser-1.1.4-10.el7.x86_64 yum-3.4.3-125.el7.noarchyum-rhn-plugin-2.0.1-5.el7.noarch --nodeps
```
# 安装yum
```
> rpm -Uvh yum-3.4.3-154.el7.centos.noarch.rpm yum-metadata-parser-1.1.4-10.el7.x86_64.rpm python-urlgrabber-3.10-8.el7.noarch.rpm yum-plugin-fastestmirror-1.1.31-42.el7.noarch.rpm yum-utils-1.1.31-42.el7.noarch.rpm  python-kitchen-1.1.1-5.el7.noarch.rpm python-chardet-2.2.1-1.el7_1.noarch.rpm
```
# 编辑 CentOS6-Base.repo
```
vi CentOS6-Base.repo 文件
```

# CentOS-Base.repo
```
下载链接：https://blog.csdn.net/u010454030/article/details/78807281
```

# 复制资源库文件至yum目录
```
> cp CentOS-Base.repo /etc/yum.repos.d/
```
# 清除缓存
```
>yum clean all
```
# 建立yum 缓存
```
>yum makecache

```
![logo](）Red-Hat-升级-rpm软件、YUM软件（root用户）/11278290-d31d27becb290491.png)
title: (二）Red-Hat-升级-rpm软件、YUM软件（root用户）
author: wen
tags:
  - linux
  - rpm
  - yum
categories:
  - linux
date: 2018-05-08 15:30:00
---
以下操作是root用户

1 [oracle VM安装red hat liunx](/2018/05/08/）oracle-VM安装red-hat-liunx/)

rpm 下载链接：http://mirrors.163.com/centos/7/os/x86_64/Packages/

# 查看版本
```
> rpm version
```
![logo](Red-Hat-升级-rpm软件、YUM软件（root用户）/11278290-76fc51b36f51535f.png)

# 升级rpm
```
>rpm -Uvh rpm-4.11.3-25.el7.x86_64.rpm  rpm-build-libs-4.11.3-25.el7.x86_64.rpm rpm-libs-4.11.3-25.el7.x86_64.rpm rpm-python-4.11.3-25.el7.x86_64.rpm --force -i  
```
![logo](Red-Hat-升级-rpm软件、YUM软件（root用户）/11278290-2b0c840c69622e92.png)
# 删除旧的yum
```
> rpm -e yum-metadata-parser-1.1.4-10.el7.x86_64 yum-3.4.3-125.el7.noarch yum-rhn-plugin-2.0.1-5.el7.noarch --nodeps
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
# CentOS-Base.repo
#
# The mirror system uses the connecting IP address of the client and the
# update status of each mirror to pick mirrors that are updated to and
# geographically close to the client.  You should use this for CentOS updates
# unless you are manually picking other mirrors.
#
# If the mirrorlist= does not work for you, as a fall back you can try the 
# remarked out baseurl= line instead.
#
#
 
[base]
name=CentOS-7 - Base - mirrors.aliyun.com
failovermethod=priority
baseurl=http://mirrors.aliyun.com/centos/7/os/$basearch/
        http://mirrors.aliyuncs.com/centos/7/os/$basearch/
#mirrorlist=http://mirrorlist.centos.org/?release=7&arch=$basearch&repo=os
gpgcheck=1
gpgkey=http://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-7
 
#released updates 
[updates]
name=CentOS-7 - Updates - mirrors.aliyun.com
failovermethod=priority
baseurl=http://mirrors.aliyun.com/centos/7/updates/$basearch/
        http://mirrors.aliyuncs.com/centos/7/updates/$basearch/
#mirrorlist=http://mirrorlist.centos.org/?release=7&arch=$basearch&repo=updates
gpgcheck=1
gpgkey=http://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-7
 
#additional packages that may be useful
[extras]
name=CentOS-7 - Extras - mirrors.aliyun.com
failovermethod=priority
baseurl=http://mirrors.aliyun.com/centos/7/extras/$basearch/
        http://mirrors.aliyuncs.com/centos/7/extras/$basearch/
#mirrorlist=http://mirrorlist.centos.org/?release=7&arch=$basearch&repo=extras
gpgcheck=1
gpgkey=http://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-7
 
#additional packages that extend functionality of existing packages
[centosplus]
name=CentOS-7 - Plus - mirrors.aliyun.com
failovermethod=priority
baseurl=http://mirrors.aliyun.com/centos/7/centosplus/$basearch/
        http://mirrors.aliyuncs.com/centos/7/centosplus/$basearch/
#mirrorlist=http://mirrorlist.centos.org/?release=7&arch=$basearch&repo=centosplus
gpgcheck=1
enabled=0
gpgkey=http://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-7
 
#contrib - packages by Centos Users
[contrib]
name=CentOS-7 - Contrib - mirrors.aliyun.com
failovermethod=priority
baseurl=http://mirrors.aliyun.com/centos/7/contrib/$basearch/
        http://mirrors.aliyuncs.com/centos/7/contrib/$basearch/
#mirrorlist=http://mirrorlist.centos.org/?release=7&arch=$basearch&repo=contrib
gpgcheck=1
enabled=0
gpgkey=http://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-7

```

# 复制资源库文件至yum目录
```
> cp CentOS6-Base.repo /etc/yum.repos.d/
```
# 清除缓存
```
>yum clean all
```
# 建立yum 缓存
```
>yum makecache

```
![logo](Red-Hat-升级-rpm软件、YUM软件（root用户）/11278290-d31d27becb290491.png)

# 再次更新yum
```
yum update yum
```

yum install yum-rhn*

yum update redhat*
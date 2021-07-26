title: （一）oracle VM安装red hat liunx
author: wen
tags:
  - red hat Linux
categories: []
date: 2018-05-08 15:01:00
---
软件：Oracle VM VirtualBox

网络桥接物流网卡：

# 1打开Oracle VMVirtualBox 点击新建按钮
![logo](）oracle-VM安装red-hat-liunx/11278290-8cae3ccf049eb036.png)
# 2 点击 下一步 按钮
名称 输入：jenkins

类型：Linux

操作系统：Red Rat(64 bit)
![logo](）oracle-VM安装red-hat-liunx/11278290-d23c40773d84371f.png)

# 3 内存 默认1G，点击 下一步
![logo](）oracle-VM安装red-hat-liunx/11278290-22f082cba471a8bb.png)

# 4 选择 现在创建虚拟硬盘 ，点击 创建 按钮
![logo](）oracle-VM安装red-hat-liunx/11278290-6a1302edff3ebf2e.png)

# 5 默认选项，点击 下一步
![logo](）oracle-VM安装red-hat-liunx/11278290-8baeb9eb905c71a2.png)

# 6 选择 动态分配 ，点击 下一步 按钮
![logo](）oracle-VM安装red-hat-liunx/11278290-7adf8cc44c0ee846.png)

# 7  设置硬盘大小:40G ，后点击创建按钮
![logo](）oracle-VM安装red-hat-liunx/11278290-4e335970ba06000f.png)

# 8 点击设置按钮
![logo](）oracle-VM安装red-hat-liunx/11278290-f9a8280cb4192f6b.png)

# 9 选择 网卡 选项，选择 桥接网卡 方式，选择存储选项
![logo](）oracle-VM安装red-hat-liunx/11278290-24e0f2beb6f3331e.png)

# 10 进行选择ISO镜像，点击图中 红色部分
![logo](）oracle-VM安装red-hat-liunx/11278290-0322ebd8568ed1e0.png)

# 11 点击选择 磁盘
![logo](）oracle-VM安装red-hat-liunx/11278290-82234f6e704f75f6.png)

# 12 在弹出窗口 选择 系统镜像，选择完成 ，点击打开按钮镜像可以从https://www.linuxprobe.com/tools下载
![logo](）oracle-VM安装red-hat-liunx/11278290-e8c8a52d140c5609.png)

# 13 点击OK 按钮后
![logo](）oracle-VM安装red-hat-liunx/11278290-e22e0734301c33b6.png)

# 14 虚拟机设置完成，点击启动
![logo](）oracle-VM安装red-hat-liunx/11278290-23e3a150c783f9fb.png)

# 15 点击 启动按钮,选择InstallRed Hat Enterprise Linux 7.1-->enter
![logo](）oracle-VM安装red-hat-liunx/11278290-b20d2391b41df6dc.png)


# 16 选择语言 界面 ，点击continue
![logo](）oracle-VM安装red-hat-liunx/11278290-fe5bdba62c19c919.png)

# 17 点击network& host name
![logo](）oracle-VM安装red-hat-liunx/11278290-77b646357a8b5e8a.png)

# 18 OFF 改为ON后，点击DONE
![logo](）oracle-VM安装red-hat-liunx/11278290-5bc80879edb2e0d6.png)

![logo](）oracle-VM安装red-hat-liunx/11278290-cd397c4737115c39.png)

# 19 选择INSTALLATIONDESTINATION
![logo](）oracle-VM安装red-hat-liunx/11278290-dc140d95517fb4de.png)

# 20 点击Done,默认设置
![logo](）oracle-VM安装red-hat-liunx/11278290-44f8decd9c26bb76.png)
# 21 点击DATE&TIME
![logo](）oracle-VM安装red-hat-liunx/11278290-62b4d6ecfa01455b.png)

# 22 Region 改为Asia ,City:上海，networkTime=ON，后点击Done

![logo](）oracle-VM安装red-hat-liunx/11278290-5b08e271ee6f5d60.png)
![logo](）oracle-VM安装red-hat-liunx/11278290-f36a34e0fbdd4dbf.png)
# 23 点击BeginInstallation
![logo](）oracle-VM安装red-hat-liunx/11278290-a75186d85c19ec0b.png)
# 24 点击ROOT PASSWORD 进行设置ROOT 密码
![logo](）oracle-VM安装red-hat-liunx/11278290-26a6a241112a9517.png)
# 25 输入ROOT 密码点击Done
![logo](）oracle-VM安装red-hat-liunx/11278290-7a2a749fed3fa8f9.png)
# 26 耐心等候安装完成 如下图，点击REBOOT
![logo](）oracle-VM安装red-hat-liunx/11278290-1df5c789751f5083.png)
# 27 重启完成，如下图，表示安装成功，使用ROOT用户进行登录
![logo](）oracle-VM安装red-hat-liunx/11278290-1edb28581c216a0d.png)
# 28 输入ip addr 查看当前虚拟机的ip
正常：

![logo](）oracle-VM安装red-hat-liunx/11278290-92aec236bf64d15a.png)


不正常：
![logo](）oracle-VM安装red-hat-liunx/11278290-105894cde675332f.png)

解决办法：

 cd/etc/sysconfig/network-scripts/

打开网卡文件进行设置静态IP vi ifcfg-enp0s3

首先把BOOTPROTO="dhcp"改成BOOTPROTO="static"表示静态获取，

然后在最后追加比如下面的配置：

IPADDR=192.168.1.33

NETMASK=255.255.255.0

GATEWAY=192.168.1.1

DNS1=202.96.134.33

保存后，

运行systemctl restart network

再次查看ip addr
![logo](）oracle-VM安装red-hat-liunx/11278290-cc3760aa781c68ef.png)

#  29 使用SecureCRT 进行连接虚拟机
![logo](）oracle-VM安装red-hat-liunx/11278290-01decabbbeebf0d4.png)
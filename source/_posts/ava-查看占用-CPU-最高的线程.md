title: '[Java] 查看占用 CPU 最高的线程'
author: wen
tags:
  - java
categories:
  - java
date: 2019-11-13 11:15:00
---
# 步骤
* top 命令找出应用 pid-app
* top -Hp <pid-app> 命令找出线程 pid-thread
* printf '%x\n' <pid-thread> 命令将线程 pid 转换成 16 进制 pid-thread-hex
* jstack <pid-app> | grep 'pid-thread-hex' 命令查看线程信息
# 示例
* 1.top 命令找出应用 pid-app
![logo](ava-查看占用-CPU-最高的线程/1.png)

应用 pid 为8790

* top -Hp <pid-app> 命令找出线程 pid-thread
![logo](ava-查看占用-CPU-最高的线程/2.png)

线程 pid 为9702

* printf '%x\n' <pid-thread> 命令将线程 pid 转换成 16 进制 pid-thread-hex
![logo](ava-查看占用-CPU-最高的线程/3.png)

* jstack <pid-app> | grep 'pid-thread-hex' 命令查看线程信息

![logo](ava-查看占用-CPU-最高的线程/4.png)
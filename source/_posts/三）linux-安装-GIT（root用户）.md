title: （三）linux 安装 GIT（root用户）
author: wen
tags: []
categories:
  - git
date: 2018-05-08 15:52:00
---
链接：
[oracle VM安装red hat liunx](/2018/05/08/）oracle-VM安装red-hat-liunx/)

 [升级RPM和YUM 包](/2018/05/08/）Red-Hat-升级-rpm软件、YUM软件（root用户）/)

# 安装GIT（root用户）
yum install git

# 1 创建用户
```
>useradd test
```

# 2 设置密码
```
>passwd test
```

# 3 创建目录
```
>mkdir -p /repo
```
# 4 初始化git 库
```
>git init --shared /repo/code
```
# 5 进入目录：
```
>cd /repo/code
```
# 6 git 忽略设置
```
>git config receive.denyCurrentBranch"ignore"
```
# 7 设置 邮件
```
>git config --global user.email  123@126.com
```
# 8 设置 用户名
```
>git config --global user.name "test"
```
# 9 添加README.md
```
>touch ./Readme.md
```

# 10 git add文件
```
>git add ./Readme.md
```

# 11 提交代码
```
>git commit -m "init"./Readme.md
```

# 12 设置权限
```
> chmod -R og+w /repoE
```

# 13 下载代码
```
git clone ssh://test@192.168.137.8/repo/code
```
# 14 添加文件
```
git add
```

# 15 提交文件
```
git commit 
```
# 16 推送代码
```
git push origin master
```
title: Hexo搭建githubpage博客
tags:
  - hexo
  - githubpage
  - hexo-admin
categories: hexo
date: 2017-02-20 10:01:00
---
<Excerpt in index | 首页摘要>
这是一篇很详尽的独立博客搭建教程 里面介绍了github、Hexo和hexo-admin设置等过程<!-- more -->
<The rest of contents | 余下全文>
## 系统环境配置
要使用Hexo，需要在你的系统中支持Nodejs以及Git，如果还没有，那就开始安装吧！

### 安装Node.js
下载:[Node.js](https://nodejs.org/en/)
参考地址：[安装Node.js](http://www.runoob.com/nodejs/nodejs-install-setup.html)

### 安装Git
下载地址：[git](http://git-scm.com/download/)

### 安装hexo

``` bash
$ cd d:/hexo
```

``` bash
$ npm install hexo-cli -g
```

``` bash
$ hexo init blog
```

``` bash
$ cd blog
```

``` bash
$ npm install
```

``` bash
$ hexo s -g #生成预览
```

$ hexo g == hexo generate  生成静态文件，会在当前目录下生成一个新的叫做public的文件夹
$ hexo s == hexo server 启动本地web服务，用于博客的预览
$ hexo d == hexo deploy 部署播客到远端（比如github, heroku等平台）

现在我们打开http://localhost:4000/ 已经可以看到一篇内置的blog了。
![logo](hexo git/1.jpeg)


目前我安装所用的本地环境如下：(可以通过hexo -v查看)
hexo-cli: 1.0.2
os: Windows_NT 6.1.7601 win32 x64
http_parser: 2.7.0
node: 7.5.0
v8: 5.4.500.48
uv: 1.10.2
zlib: 1.2.8
ares: 1.10.1-DEV
modules: 51
openssl: 1.0.2k
icu: 58.2
unicode: 9.0
cldr: 30.0.3
tz: 2016j


## Hexo主题设置
这里以主题yilia为例进行说明。

### 安装主题 (在blog目录下)
``` bash
$ hexo clean
```
``` bash
$ git clone https://github.com/litten/hexo-theme-yilia.git themes/yilia
```

### 启用主题

修改Hexo目录下的_config.yml配置文件中的theme属性，将其设置为yilia。

### 更新主题
``` bash
$ cd themes/yilia
```
``` bash
$ git pull
```
``` bash
$ hexo s -g
```
现在打开http://localhost:4000/ ，会看到我们已经应用了一个新的主题。
![logo](hexo git/1.jpeg)


## Github Pages设置

### 什么是Github Pages

GitHub Pages 本用于介绍托管在GitHub的项目，不过，由于他的空间免费稳定，用来做搭建一个博客再好不过了。
每个帐号只能有一个仓库来存放个人主页，而且仓库的名字必须是username/username.github.io，这是特殊的命名约定。你可以通过http://username.github.io 来访问你的个人主页。
这里特别提醒一下，需要注意的个人主页的网站内容是在master分支下的。

在这里我创建了一个github repo叫做 smallwenzi.github.io. 创建完成之后，需要有一次提交(git commit)操作，然后就可以通过链接http://smallwenzi.github.io/ 访问了。

### 配置SSHKey
我们如何让本地 git 项目与远程的 GitHub 建立联系呢？用 SSH keys。

#### 检查 SSH keys的设置
首先我们需要检查你电脑上现有的 ssh key：
~/.ssh目录是否存在
若不存在
```
ssh-keygen -t rsa -C "邮件地址@youremail.com"

注意 1: 此处的邮箱地址，你可以输入自己的邮箱地址；
注意2: 此处的「-C」的是大写的「C」
然后系统会要你输入密码：
Enter passphrase (empty for no passphrase):<输入加密串>
Enter same passphrase again:<再次输入加密串>
在回车中会提示你输入一个密码，这个密码会在你提交项目时使用，如果为空的话提交项目时则不用输入。这个设置是防止别人往你的项目里提交内容。
注意：输入密码的时候没有 * 字样的，你直接输入就可以了。

最后看到这样的界面，就成功设置ssh key了：
```
 ![logo](hexo git/5.png)
### 添加 SSH Key 到 GitHub
在本机设置 SSH Key 之后，需要添加到 GitHub上，以完成 SSH 链接的设置。

1、打开本地 id_rsa.pub 文件（ 参考地址 C:\Documents and Settings\Administrator.ssh\id_rsa.pub）。此文件里面内容为刚才生成的密钥。如果看不到这个文件，你需要设置显示隐藏文件。准确的复制这个文件的内容，才能保证设置的成功。

2、登陆 GitHub 系统。点击右上角的 Account Settings--->SSH KEY AND GPG KEY  ---> new ssh key
3、把你本地生成的密钥复制到里面（ key 文本框中）， 点击 add key 就ok了

## 部署Hexo到Github Pages

这一步恐怕是最关键的一步了，让我们把在本地web环境下预览到的博客部署到github上，然后就可以直接通过http://smallwenzi.github.io/访问了。
首先需要明白所谓部署到github的原理。
1之前步骤中在Github上创建的那个特别的repo（smallwenzi.github.io）一个最大的特点就是其master中的html静态文件，可以通过链接http://smallwenzi.github.io来直接访问。
2Hexo -g 会生成一个静态网站（第一次会生成一个public目录），这个静态文件可以直接访问。
3需要将hexo生成的静态网站，提交(git commit)到github上。
明白了原理，怎么做自然就清晰了。

### 使用hexo deploy部署
hexo deploy可以部署到很多平台，具体可以参考这个[链接](https://hexo.io/docs/deployment.html). 如果部署到github，需要在配置文件_config.xml中作如下修改：
deploy:
  type: git
  repo: git@github.com:smallwenzi/smallwenzi.github.io.git
  branch: master
  
 ![logo](hexo git/2.png)
然后在命令行中执行 (在blog目录下)
``` bash
$ npm install hexo-deployer-git --save
```
``` bash
$ hexo d -g
```
即可完成部署。现在打开https://smallwenzi.github.io/ ，

简单解释一下，hexo generate生成public文件夹下的新内容，然后将其拷贝至smallwenzi.github.io的git目录下，然后使用git commit命令提交代码到smallwenzi.github.io这个repo的master branch上。


### Hexo 主题配置

每个不同的主题会需要不同的配置，主题配置文件在主题目录下的_config.yml。
以我们使用的yilia主题为例，其提供如下的配置项（theme\yilia_config.yml）：
[参考](https://github.com/litten/hexo-theme-yilia).


## Hexo 使用本地图片
首先确认 _config.yml 中有 post_asset_folder:true 。
在 hexo 目录，执行

npm install https://github.com/CodeFalling/hexo-asset-image --save
假设在

MacGesture2-Publish
├── apppicker.jpg
├── logo.jpg
└── rules.jpg
MacGesture2-Publish.md

![logo](hexo git/3.png)

## Hexo 后台管理配置
在blog 跟目录下
``` bash
$ npm install --save hexo-admin 
```
``` bash
$ hexo server -d
```
如下图：
![logo](hexo git/4.png)
[参考](https://github.com/jaredly/hexo-admin)

## github后台绑上自己的域名

### 申请一个域名
我用的新网买的域名（yanwentan.xyz），买好之后进入域名控制台可以看到自己买的域名。
![logo](hexo git/7.png)
设置域名解析记录，注意记录类型设置为CNAME，记录值为刚才github上仓库的名字，注意没有https://。
![logo](hexo git/8.png)

为了能够从别的域名跳转到github页面，还需要在github中进行设置：
在博客source目录新增CNAME 文件 
文件内容 添加自己的域名：blog.yanwentan.xyz

这样 通过hexo d -g 部署 就会 设置 域名 显示

## 免费申请https
https://dash.cloudflare.com 注册一个账号，注册成功后
* 新增站点yanwentan.xyz
![logo](hexo git/10.png)
* 修改dns映射
![logo](hexo git/12.png)
NS	gemma.ns.cloudflare.com
NS	henrik.ns.cloudflare.com
生效时间24-48小时 
* 添加cname记录
添加 blog 映射：smallwenzi.github.io
![logo](hexo git/11.png)
设置完成使用 https://blog.yanwentan.xyz/ 访问
* 点击上图ssl\TLS  设置您的 SSL/TLS 加密模式为 灵活
![logo](hexo git/13.png)
* 点击上图 边缘服务器 进行设置 https
![logo](hexo git/15.png)
* 点击 analytics 查看网站访问流量
![logo](hexo git/16.png)


## 注意事项（避雷）
* 创建.cn/.com等域名可能需要进行实名认证才能正确解析，这在域名控制器中会有提示，若域名解析成功，控制器中会显示DNS服务器地址表示正常服务：


* 在设置完成后不会立刻生效，需要等待一段时间（10~15分钟），这期间尽量不要因为无法访问而反复更改设置，会因为设置生效有时延而变得更加混乱。

* 要注意浏览器自动缓存问题，在第一次输入后，如果不清除缓存，就算做了更改，浏览器还是会用上一次的DNS解析结果尽管这可能是错误的，所以如果更改设置后还是和原来一样的错误结果，可以试试清除浏览器缓存。
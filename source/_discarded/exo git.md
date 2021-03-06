title: Hexo搭建githubpage博客
tags:
  - hexo githubpage
categories: []
date: 2017-02-20 10:01:00
---

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

参考：[配置SSH keys](http://www.jianshu.com/p/05289a4bc8b2)

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

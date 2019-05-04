title: solr教程（一）
author: wen
tags:
  - solr
categories: []
date: 2018-10-13 13:30:00
---
# 1.为什么选择Apache Solr
Apache Solr是一个功能强大的搜索服务器，它支持REST风格API。Solr是基于Lucene的，Lucene 支持强大的匹配能力，如短语，通配符，连接，分组和更多不同的数据类型。它使用 Apache Zookeeper特别针对高流量进行优化。Apache Solr提供各式各样的功能，我们列出了部分最主要的功能。

* 先进的全文搜索功能。
* XML，JSON和HTTP - 基于开放接口标准。
* 高度可扩展和容错。
* 同时支持模式和无模式配置。
* 分页搜索和过滤。
* 支持像英语，德语，中国，日本，法国和许多主要语言
* 丰富的文档分析。

# 安装solr
服务器是Linux
要首先让从以下位置下载最新版本的Apache Solr：
http://lucene.apache.org/solr/downloads.html
在撰写本文时，可用的稳定版本是7.5.0。
一旦Solr的zip文件下载将它解压缩到一个文件夹。提取的文件夹看起来像下面。
![logo](solr教程（一）/1.png)

Solr的文件夹

bin文件夹中包含用来启动和停止服务器的脚本。example 文件夹包含几个示例文件。我们将使用其中的一种，以说明Solr如何索引数据。server 文件夹包含logs 文件夹，所有的Solr的日志都写入该文件夹。这将有助于索引过程来检查任何错误日志。在sever文件夹下的Solr文件夹包含不同的集合或核心（core/collection）。对于各集合或核心的配置和数据都存储在相应的集合或核心文件夹。

Apache Solr带有一个内置的Jetty服务器。但在我们开始之前，我们必须验证JAVA_HOME已经配置。

我们可以使用命令行脚本启动服务器。让我们去solr的bin目录，从命令提示符输入出以下命令
./bin/solr start
这将启动下的默认端口8983 Solr的服务器。

现在，我们可以在浏览器中打开以下网址和验证，我们的Solr的实例正在运行。solr的管理工具的细节超出了示例的范围。

http://localhost:8983/Solr/
![logo](solr教程（一）/2.png)


title: Solr的主从模式Master-Slave
author: wen
tags:
  - solr
  - solr-master
  - solr-slave
categories:
  - solr
date: 2018-10-14 15:25:00
---
# 摘要：
如今，为了提高Solr的搜索速度，使其具有很好的容灾能力，往往会配置SolrCloud，但在Solr4之前，还有一种很流行的方式，Master-Slave模式，为什么要提及这种方式，因为我们公司目前用的就是这种方式。

# 引入Master-Slave
Solr在查询的时候，特别忌讳进行写操作，因为它是IO阻塞型的。现在的流行的Elasticsearch就对此有很好的改进。在引入Master-Slave以后，将读写分配到不同的服务器上，你可以使用master来做索引，然后使用slaves来做查询。
1.在多台服务器上分别搭建好可以独立运行的Solr，参见[这里](https://smallwenzi.github.io/2018/10/13/lr%E6%95%99%E7%A8%8B/)
2.指定其中的一台为Master，只需要在SolrConifg.xml中配置：
```
   <requestHandler name="/replication" class="solr.ReplicationHandler">
        <lst name="master">
            <str name="enable">${enable.master:true}</str>
            <!--
                Create a backup after 'optimize'. Other values can be 'commit', 'startup'.
                It is possible to have multiple entries of this config string.
                Note that this is just for backup, replication does not require this.
            -->
            <str name="backupAfter">optimize</str>
            <!-- Replicate on 'commit'. 'startup' and 'optimize' are also the valid values for replicateAfter. -->
            <str name="replicateAfter">commit</str>
            <!-- If configuration files need to be replicated give the names here, separated by comma -->
            <str name="confFiles">schema.xml,dict.txt,synonyms.txt</str>
            <str name="commitReserveDuration">00:00:10</str>
        </lst>
        <int name="maxNumberOfBackups">2</int>
           </requestHandler>
```
3.指定其他多有的服务为Slave，只需要分别SolrConifg.xml配置：
```
  <requestHandler name="/replication" class="solr.ReplicationHandler" >
    <lst name="slave">
        <str name="enable">${enable.slave:true}</str>
        <str name="masterUrl">http://192.168.1.102:8983/solr/jcg</str>
        <str name="pollInterval">00:00:10</str>
        <str name="httpConnTimeout">5000</str>
        <str name="httpReadTimeout">10000</str>
     </lst>
  </requestHandler>
```
4.重启所有的master-slave服务即可
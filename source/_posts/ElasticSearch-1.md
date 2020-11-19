---
title: ElasticSearch(一)
date: 2020-11-19 17:14:20
tags:
-ElasticSearch
-Java
---

## Es是什么？

```
ElasticSearch是一款基于Lucene的全文搜索引擎。
```

### 什么是全文搜索引擎？

> 百度：全文搜索引擎就是通过从互联网上提取的各个网站的信息（以网页文字为主）而建立的数据库中，检索与用户查询条件匹配的相关记录，然后按一定的排列顺序将结果返回给用户。

Es是对Lucene的封装，同时也是Lucene的分布式解决方案。将绝大多数的Lucene功能封装使其更容易被使用。Github、Stackoverflow等网站都在使用ElasticSearch作为搜索引擎。同时其优秀的分布式作业管理机制使其越来越被开发人员所喜爱。

## Es安装

```
值得注意的是Es是依赖于Java环境的，当运行在Java 8 环境时会建议使用Java 11，因为在后续的版本中将取消对Java 8 的支持。
```

### 一、安装方式

> 目前官网不再提供源码下载，但是还是建议使用源码的方式进行安装，因为后续修改配置文件会比较方便。

```
//linux环境

//获取源码包
$ wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-5.6.9.tar
//解压源码包
$ tar -zxvf elasticsearch-5.6.9.tar
//进入源码主目录
$ cd elasticsearch-5.6.9/
```

至此，Es的安装就已经全部安装完毕了。

### 二、配置

打开./config/elasticsearch.yml文件

```
vim ./config/elasticsearch.yml

//修改如下内容

cluster.name: 集群名称

node.name: 节点名称

path.data: 数据存储位置（目录位置，不需要创建文件）

path.logs: 日志存储位置（同上）

network.host: 绑定的ip（默认是127.0.0.1，本机访问，可以改为0.0.0.0使其能够在公网访问）

discovery.zen.ping.unicasts.hosts: 提供其他 Elasticsearch 服务节点的单点广播发现功能。配置集群中基于主机 TCP 端口的其他 Elasticsearch 服务的逗号分隔列表。
例如：discovery.zen.ping.unicast.hosts="localhost:9300,localhost:9301,localhost:9302
```

基本的配置如上所示。

### 三、启动

```
//启动命令（Es主目录下）
./bin/elasticsearch
```

> 如果是以root的身份去启动程序，会有报错。需要更换一个身份用户再次启动，同时将Es所在的文件夹权限设置为该用户可以访问和读写，否则会有报错出现。最简单的方法就是将Es文件夹及其子文件权限设置为777。重新执行指令之后会有日志信息打印出来，如果一切顺利，就可以访问http://ip地址:9200 来体验一下了。

```
//访问服务器所返回的JSON数据
{
  "name" : "节点名称",
  "cluster_name" : "集群名称",
  "cluster_uuid" : "集群ID",
  "version" : {
    "number" : "Es版本号",
    "build_hash" : "877a590",
    "build_date" : "2018-04-12T16:25:14.838Z",
    "build_snapshot" : false,
    "lucene_version" : "6.6.1"
  },
  "tagline" : "You Know, for Search"
}
```

如果不顺利，根据报错信息。

> 如果报错关于内存，请检查服务器的内存大小，如果小于2G，不建议在此服务器部署Es。即使程序运行正常，无端出现kill情况的也是由于内存过小导致的。

> 报错出现程序创建文件受到限制，需要修改该用户的最大创建文件数量（百度搜索即可）
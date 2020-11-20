---
title: Centos7部署MongoDB
date: 2019-08-11 17:20:07
tags:
- MongoDB
---

#### 安装MongoDB

> **下载源码包**
> `mkdir mongo && cd mongo`新建一个文件夹用来存放源码文件
> 下载：
> 1、在Linux在终端执行`weget 'mongoDB下载地址'`
> 2、在本地下载完成后，使用FTP工具上传至Linux服务器
> 3、关注文末微信公众号，回复“MongoDB”获取网盘链接^^
> `tar zxvf 包文件.tar`解压文件。

> **配置文件**
> 进入mongodb解压文件夹，编辑mongo.conf文件（如果没有自己创建也可以）

```
//编辑文件
vim mongo.conf
//写入如下文本

#port 端口号
port=27017

#dbpath 数据库存储文件目录
dbpath=/home/mongo/data

#logpath 日志路径
logpath=/home/mongo/logs/mongodb.log

#logappend 日志追加形式  false:重新启动覆盖文件
logappend=true

#fork 后台启动
fork=true

#开启副本集
replSet=[副本集名称]

#绑定ip
bind_ip=0.0.0.0
```

> **启动**

```
cd ./bin

//启动
./mongo --config [配置文件地址]

//进入客户端
./mongo --port 27017

//显示如下内容 PRIMARY>表示当前结点为主节点，在此节点的写操作和修改操作将会同步到副本节点
MongoDB shell version v4.0.13
connecting to: mongodb://127.0.0.1:27017/?gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("89d46323-09ab-47c1-a37a-95f20cd030bd") }
MongoDB server version: 4.0.13
[副本集名称]:PRIMARY>

//创建初始化配置文件，如果是单机安装，可以省略。
[副本集名称]:PRIMARY>
config = {
"_id":"副本集名称",
"members":[
  {"_id":0,"host":"192.168.198.224:27017"},#可以是单机的不同端口之间按进行集群
  {"_id":1,"host":"192.168.198.225:27017"},
  {"_id":2,"host":"192.168.198.226:27017",arbiterOnly:true}
]
}

//执行初始化过程，没有配置文件可以省略参数config
[副本集名称]:PRIMARY>rs.initiate(config);

//查看集群状态
[副本集名称]:PRIMARY>rs.status();

//创建数据库
use test

//插入数据,testdb为集合名称，可自动创建
db.testdb.insert({"name":"胡图图"});

//打开其他节点的客户端，此时其他节点的数据已经同步更新，因为mongodb默认是从主节点读写数据的，副本节点上不允许读，需要设置副本节点可以读。
db.getMongo().setSlaveOk();#

//正常读取
db.testdb.find();
```

打开mongoDB副本集是为了以后的mongoDB 与 ElasticSearch 进行数据同步做准备。
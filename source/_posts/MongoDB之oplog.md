---
title: MongoDB之oplog
date: 2019-10-09 17:21:24
tags:
- MongoDB
---

> **初识oplog**
> 在进行mongoDB与Es数据同步时，被告知只有开启MongoDB副本集才可以进行数据同步，副本集最重要的一点就是节点之间同步数据的以及MongoDB与Es同步数据，依靠的就是oplog。

> **认识oplog**
> oplog 是 MongoDB 主从复制层面的一个概念，通过 oplog 来实现复制集节点间数据同步，客户端将数据写入到 Primary，Primary 写入数据后会记录一条 oplog，Secondary 从 Primary（或其他 Secondary ）拉取 oplog 并重放，来确保复制集里每个节点存储相同的数据。
> oplog 在 MongoDB 里是一个普通的 capped collection，对于存储引擎来说，oplog只是一部分普通的数据而已。——MongoDB中文社区

> **Tips**

- capped collection 为一个固定大小集合，oplog存在覆盖的情况，实际上这种情况也是允许的，也不能一直保存这些“操作日志”。所以设置oplog所在集合的大小是一件十分重要的事情，设置过小会导致数据备份丢失的危险；设置过大会浪费存储空。一般默认的是disk空间的5%（1G~50G）。可以通过–oplogSize参数来设置大小。
- oplog中的同一条记录执行多次的结果和执行一次的结果是一样的。

oplog是MongoDB与Es数据同步的基础，mongo-connrctor是利用oplog向Es中写入数据的。
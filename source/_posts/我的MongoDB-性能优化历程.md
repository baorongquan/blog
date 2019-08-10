---
title: 我的MongoDB 性能优化历程
date: 2019-08-10 16:15:34
tags: [mongo]
categories: programming
---

# 慢查询
## 慢查询设置
例如:要查看超过200毫秒的操作可如下设置
``` MongoDB
db.setProfilingLevel(1, 200)
```
刚开始使用MongoDB的时候没有去考虑性能问题,只是简单用来完成业务功能.伴随着数据量的增长,页面查询所需要的时间越来越长.于是开始思考为什么查询会变慢.顺其自然的就开始查找慢查询日志,通过慢查询日志可看到那些操作耗时多久.可以帮助优化代码或调整索引来进行调优.默认情况下慢查询日志为关闭状态.通过以下命令开启
``` MongoDB
db.setProfilingLevel(level, options)
```
level分为0:关闭,1:过滤出比设置的时间(由options设置)更久的查询,2:显示所有操作
opitons分为slowms:默认100毫秒和level为1配合使用,含义为设置一个慢查询的时间阈值;sampleRate采样率,个人暂未使用过,不做讨论.

## 慢查询查看
慢查询设置完后如果有超过设置时间的操作,则在system.profile里面可以查看.命令如下
``` MongoDB
db.system.profile.find()
```
会有类似如下的输出,我删除了不重要的显示,可以自行试验查看输出:
```
{
        "op" : "query",
        "ns" : "test.loginrecord",
        "query" : {

        },
        "millis" : 300,
        .
        .
        .
}
```
几个主要字段的含义:
* op:进行的是什么操作,上面输出进行的操作为查询
* ns:对那个库那个集合进行的操作,上面的输出是对test库的loginrecord集合进行的操作
* query:查询条件,因为我这里没有查询条件所以这里是空
* millis:操作耗时,上面表示用时300毫秒

知道了这些字段的含义后可以对慢查询日志进行过滤输出,
例如要查看对test库user集合的慢查询可以
```
db.system.profile.find({ns:"test.user"}).pretty()
```
pretty是为了让输出更方便查看.
要过耗时最久的查询可以
```
db.system.profile.find().sort({millis:-1}).limit(5).pretty()
```
sort表示按耗时降序排序,limit表示只输出5条.

注意,开启慢查询会对性能有一定消耗,所以线上环境查看完后需要进行关闭
``` MongoDB
db.setProfilingLevel(0)
```
然后删除慢查询日志数据,必须关闭后才能删除.
```
db.system.profile.drop()
```
到此就是慢查询的基本设置和使用了,由于这里使用的例子比较简单,所以慢查询输出数据也相对简单,下面会在添加了索引的状态下丰富慢查询输出.

# Index

# Explain
## stage
## executionStats
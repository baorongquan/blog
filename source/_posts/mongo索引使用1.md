---
title: mongo索引使用1
date: 2017-10-14 15:43:49
tags: [mongo]
categories: 数据库
---

在数据库做查询时,适当的索引可以提高查询的效率，避免全表扫描。
mongo建立索引的时候需要指定索引为升序1或降序-1,对于单个字段建立索引时这个排序规则并不重要，
因为mongodb可以沿任一方向遍历索引。
# 索引创建
假设我有一个loginrecord集合(下面是golang 定义),有两个字段，string类型的username和时间类型的logintime
``` golang
type LoginRecord struct {
	UserName string `bson:"username"`
	LoginTime time.Time `bson:"logintime"`
}

```
记录用户和该用户的登陆记录，现在对用户名建立索引;
``` javascript
db.loginrecord.createIndex({username:1})
```
# 索引查看
``` javascript
db.loginrecord.getIndexes()
```
此时会看到有两个索引，一个是我们刚刚建立的username，另一个是mongodb默认以_id创建的索引。
```
> db.loginrecord.createIndex({username:1})
> db.loginrecord.getIndexes()
[
        {
                "v" : 1,
                "key" : {
                        "_id" : 1
                },
                "ns" : "test.loginrecord",
                "name" : "_id_"
        },
        {
                "v" : 1,
                "key" : {
                        "username" : 1
                },
                "ns" : "test.loginrecord",
                "name" : "username_1"
        }
]

```
不过我更喜欢用
``` javascript
> db.loginrecord.stats()
{
        "ns" : "test.loginrecord",
        "count" : 1,
        "size" : 72,
        "avgObjSize" : 72,
        "storageSize" : 4096,
        "numExtents" : 1,
        "nindexes" : 2,
        "lastExtentSize" : 4096,
        "paddingFactor" : 1,
        "systemFlags" : 1,
        "userFlags" : 0,
        "totalIndexSize" : 16352,
        "indexSizes" : {
                "_id_" : 8176,
                "username_1" : 8176
        },
        "ok" : 1
}

```
来查看索引，indexSizes里面的就是索引,_1表示升序，_-1表示降序

# 索引删除
``` javascript 
> db.loginrecord.dropIndex({username:1})
{ "nIndexesWas" : 2, "ok" : 1 }

```
索引删除的时候也必须指定升序还是降序，不然会找不到报错

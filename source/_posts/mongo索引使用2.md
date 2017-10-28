---
title: mongo索引使用2
date: 2017-10-15 17:15:39
tags: [mongo]
categories: 数据库
---

前面说了mongo的索引创建，查看，删除；今天主要来说下复合索引和索引使用时的注意事项。
# 复合索引
mongodb支持用户为多个域创建索引，也就是复合索引。
还是以用户登陆记录集合loginrecord为例，假设我们要查询用户登陆的每条记录，
并按登陆时间逆序显示，所以需要为username和logintime创建复合索引。

``` javascript
> db.loginrecord.createIndex({username:1,logintime:-1})
> db.loginrecord.stats()
{
        "ns" : "test.loginrecord",
        "count" : 1,
        "size" : 72,
        "avgObjSize" : 72,
        "storageSize" : 4096,
        "numExtents" : 1,
        "nindexes" : 3,
        "lastExtentSize" : 4096,
        "paddingFactor" : 1,
        "systemFlags" : 0,
        "userFlags" : 0,
        "totalIndexSize" : 24528,
        "indexSizes" : {
                "_id_" : 8176,
                "username_-1" : 8176,
                "username_1_logintime_-1" : 8176
        },
        "ok" : 1
}
```
此时我们通过
``` javascript
db.loginrecord.find({username:"zhangsan"}).sort({logintime:-1})
```
进行查询时就会用到{username:1,logintime:-1}的复合索引

# 使用注意事项
## 1、find方法和sort方法都会用到索引
例如，当我们需要查看用户的登陆记录，并将记录按用户名按字母序输出
``` javascript
db.loginrecord.find({}).sort({username:1})
```
sort方法就会用到以username为键的索引。
如果我们要查询用户名为zhangsan的登陆记录，并按登陆时间logintime逆序输出
``` javascript
db.login.record.find({username:"zhangsan"}).sort({logintime:1})
```
此时就会用到username和logintime的复合索引(注意：不是username,logintime的单独索引)。

## 2、复合索引字段的顺序很重要
复合索引创建时字段的先后顺序是很重要的，需要知道的是以{username:1,logintime:-1}和以{logintime:-1,username:1}
创建的复合索引是不同的，{username:1,logintime:-1}是先以username分组然后再在满足条件的分组里以logintime分组；
而{logintime:-1,username:1}则是先按logintime分组然后再在满足条件的分组里以username分组。
为了说明复合索引顺序的问题，我们先为loginrecord集合加一个字段online用来表示用户当前是否在线。
``` golang
type LoginRecord struct {
	UserName  string    `bson:"username"`
	LoginTime time.Time `bson:"logintime"`
	OnLine    bool		`bson:"online"`
}
```
现在假设用户登陆后online字段为true，用户登出后改为false，于是在只允许单用户登陆的情况下，loginrecord集合里面
一个用户就只可能有一条online为true的记录。假设用户量很大且登陆频繁的情况下，我们需要查找当前在线的用户，
按用户名字母序输出。这个时候就需要建立online和username两个字段的复合索引。于是我们就要考虑
> 情况1：建立{username:1,online:1}的复合索引
> 情况2：建立{online:1,username:1}的复合索引

### 情况1：建立{username:1,online:1}的复合索引
先以用户名分组，在已online分组，此时由于每个用户都存在多条登陆记录而只有一条在线记录，
此时每个username分组我们都只能找到一条记录。然后又在把每条记录以username排序，实际是先删选出来然后再排序，
索引{username:1,online:1}就没有创建的意义。

### 情况2：建立{online:1,username:1}的复合索引
先以online分组，在以username分组，此时就能很方便的从online为true的分组里找到所有记录。查询速度是
情况1所不能比的。所以根据自己的业务需要设定复合索引的字段顺序很重要。

> 注： mongodb的索引的数据是B树,感兴趣的可以自己搜索以便更好理解mongodb的索引使用

## 3、复合索引还可以支持与索引字段前缀匹配的查询
复合索引除了支持所有索引字段的匹配查询外，还支持与索引字段前缀匹配的查询。怎么理解呢?
例如上面我们建立了{username:1,logintime:-1}的索引，此时不但可以同时匹配{username:"xxx",logintime:"xxx"}
这种查询，还支持{username:"xxx"}这种查询，但不支持{logintime:"xxx"}这种查询，因为logintime不是
{username:1,logintime:-1}的前缀
``` javascript
db.loginrecord.find({username:"zhangsan",logintime:{"$gt":ISODate("2017-06-08T01:41:28.944Z")}})
db.loginrecord.find({username:"zhangsan"})
```
因此，如果你建立了{username:1,logintime:-1}，就不必在建立{username:1}或者{username:-1}这种索引，
要知道索引多了是会影响插入和更新操作的。


---
title: 我不知道的go里面的json操作
date: 2018-04-05 12:04:39
tags: [golang, json]
categories: programming
---

# 一、可以在tag里面声明把Integer，Float，Boolean类型的值直接序列化时转换为string类型返回
``` golang
{
    ID int64 `json:"id,string"`
    Name string `json:"name"`
}
```
这样在返回给前端的时候就不用自己把int64转成然后再返回给前端了，毕竟javascript不支持int64

# 二、 可以把tag写成"-", 表示忽略该字段序列化的时候忽略该字段
``` golang
{
    ID int64 `json:"id,string"`
    Name string `json:"name"`
    Age int `json:"-"`
}
```

# 三、自己实现MarshalJSON 和 UnmarshalJSON 替换掉默认的Marshal和UnMarshal方法，自己定制序列化和反序列化

需要了解更多json的高级操作可以看json的源码注释
> /src/encoding/json/encode.go

和这篇文章
> https://blog.gopheracademy.com/advent-2016/advanced-encoding-decoding/ 

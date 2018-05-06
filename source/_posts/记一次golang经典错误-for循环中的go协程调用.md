---
title: 记一次golang经典错误--for循环中的go协程调用
date: 2018-05-06 16:04:30
tags: [golang]
categories: program
---

最近在开发过程中遇到问题，追踪了很久后发现是golang的经典问题，在for循环中使用了goroutine,在goroutine中使用了for循环的参数。

# 问题现象：
在使用rabbitmq进行数据传递时，发送端在一次循环中发送了8000条id不同的数据到rabbitmq的队列中，接收端监听该队列并从rabbitmq中取数据。接收到的数据在程序中处理后写入数据库，结果发现数据中并没有写入8000条数据。最后定位原因为：在接收数据时在for循环中使用go协程，导致同时收到两条数据时，协程都是使用的后一条数据，入库因为是同一条数据，导致主键重复，插入失败，所以数据库中没有8000条数据。错误代码大致如下：
``` golang
for d := range msgs {
    go func() {
        handler(d)
    }()
}
```

# 用一个简单的程序模拟该错误为：
``` golang
package main

import (
	"fmt"
	"time"
)
func main() {
	for i := 0; i < 10; i++ {
		go func() {
			fmt.Println(i)
		}()
	}
	time.Sleep(2 * time.Second)
}
```
输出为：
> 7
>10
>10
>10
>10
>10
>10
>10
>10
>10

# 问题解析：
闭包go协程里面引用的是变量i的地址;所有的go协程启动后等待调用，在上面的协程中，部分协程很可能在for循环完成之后才被调用，所以输出结果很多都是10;<font color="yellow">正常输出最多到9哦</font>

# 解决方法一
通过参数传递数据到协程
``` golang
for i := 0; i < 10; i++ {
    go func(data int) {
        fmt.Println(data)
	}(i)
}
```
# 解决方法二
在for循环中加一个临时变量tmp，每次将i的值赋值给tmp，然后将tmp通过参数传进协程。
<font color="red">此方法可以解决不能通过参数传递数据的情况(某些第三方库不能传参数)</font>
```golang
for i := 0; i < 10; i++ {
    tmp := i
    go func(data int) {
        fmt.Println(data)
    }(tmp)
}
```
---
title: golang在recover里面发生panic
date: 2018-05-12 10:28:37
tags: [golang]
categories: programming
---
最近在排查代码的时候，发现同事有一段代码在recover里面关闭连接结果导致panic，造成程序异常退出的现象。
用一个小试例记录一下，正常代码在recover里面打印捕获到的信息为：
``` golang
package main

import "fmt"

func main() {
	for i := 0; i < 10; i++ {
		f(i)
	}
	fmt.Println("end main")
}

func f(i int) {
	defer func() {
		if r := recover(); r != nil {
			fmt.Println("Recover :", r)
			// panic(r)
		}
	}()
	if i == 8 {
		panic("panic when i == 8")
	}
	fmt.Println(i)
}
```
程序可以正常执行，输出如下：
> 0
> 1
> 2
> 3
> 4
> 5
> 6
> 7
> Recover : panic when i == 8
> 9
> end main

但是如果在recover里面的panic注释去掉，模拟产生panic，再次运行就会发生错误。
<font color=red>所以切忌不要在recover里面发生panic，如关闭不存在的连接等。</font>

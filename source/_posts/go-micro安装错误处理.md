---
title: go-micro安装错误处理
date: 2018-04-18 20:06:06
tags: [program]
categories: 配置
---

# go-micro 安装错误种的错误处理
按照github上[micro](https://github.com/micro/micro/blob/master/README.md)的教程使用非docker方式安装的错误处理
``` shell
go get -u github.com/micro/micro
```
会报关于 golang.org/x/crypto/acme/autocert 的错误。
* 情况1：由于防火墙的原因导致连接网络失败。
* 情况2：由于之前手动拷贝安装过crypto这个包会报使用一个未知的版本控制系统的错误

解决办法：
```
cd $GOPATH/src/golang.org/x/
git clone https://github.com/golang/crypto.git
```
<font color=red>注意:</font>
```
1、不要手动下载然后拷贝过去，会报上述情况2的错误
2、之前手动拷贝过crypto的话，先备份或删除再git clone
```

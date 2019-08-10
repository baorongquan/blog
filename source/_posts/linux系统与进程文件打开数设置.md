---
title: linux系统与进程文件打开数设置
date: 2019-08-08 14:28:37
tags: [linux, mongo]
categories: 配置
---

场景是这样的：小伙伴在mongodb主备环境把备机停了一段时间后重新启动，mongodb主机在同步数据的过程中出现"too many open files error", 在网上查到的说可以通过设置linux系统文件打开数来解决(服务器是Ubuntu),
然后开始一顿操作修改 /etc/security/limits.conf 进行设置
``` shell
vi /etc/security/limits.conf 添加以下内容
* soft nproc 65533
* hard nproc 65533 
* soft nofile 65533 
* hard nofile 65533 
root soft nproc 65533 
root hard nproc 65533 
root soft nofile 65533 
root hard nofile 65533
```
重启后 ulimit -a 查看系统文件打开数已修改。但是查看mongodb进程限制仍未改变
``` shell
cat /proc/pid/limits  // pid 为mongod的进程id
```
登陆mongo通过,查看mongo连接数仍未改变。
``` shell
db.runCommand({serverStatus:1}).connections
```

最后在stackoverflow 找到答案, 修改/etc/systemd/user.conf 以及 /etc/systemd/system.conf
添加以下内容
``` shell
DefaultLimitNOFILE=65535 
```
然后reboot重启后再看已生效
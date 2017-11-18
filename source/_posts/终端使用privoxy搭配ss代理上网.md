---
title: 终端使用privoxy搭配ss代理上网
date: 2017-11-18 16:54:46
tags: [网络]
categories: 配置
---

作为一名程序员，翻墙技能是必备的，但是你会发现仅仅使用浏览器翻墙师不够，有时你
还需要在终端下git个项目，但是发现由于墙的原因老是下载不好，这个时候，就需要配置下终端代理翻墙了，
当然你也可以配置各种源或是浏览器下载了在安装（如果不觉得每次都这样很麻烦的话）。
下面介绍下实现终端代理翻墙的方法。
# 一、使用前必须具备的条件
## 有ss或ssr可使用于科学上网
## 能在终端安装使用的代理http（https）请求的工具，本文使用privoxy

# 二、运行你正在使用的ss（ssr）工具，通过浏览器验证能翻墙上外网
# 三、安装privoxy
```shell
sudo apt-get install privoxy
```
安装完后修改privoxy的配置文件
```shell 
sudo vim /etc/privoxy/config
```
在750行左右找到listen-address，在下面添加两行，内容如下(注意不要加注释，也就是前面不要加#号)
```txt
listen-address  0.0.0.0:8118
forward-socks5 / localhost:1080 .
```
# 运行privoxy
修改用户主目录下的shell文件，可能是.bashrc或者.bash_profile或者.zshrc
添加内容如下
```shell
/usr/sbin/privoxy /etc/privoxy/config
export http_proxy=http://127.0.0.1:8118
export https_proxy=https://127.0.0.1:8118
```
保存退出后,source 你刚刚修改的文件
```shell
source ~/.zshrc
```
这个时候使用
```shell
wget www.google.com
```
验证下是否能获取到index.html文件，在终端无法翻墙的情况下是获取不到的,
国内网站，比如百度就可以

到此，配置结束。

# 注意
privoxy代理了终端所有的http（https）请求，有时可能你的一些操作可能不需要代理，
但是老是失败，可以通过错误信息看看是不是因为请求被代理到8118端口了，如果是，你需要
找到privoxy的程序，然后kill掉，再重试下应该就可以了。如果需要重新启用privoxy，再次
source ~/.zshrc或者service restart privoxy就可以了。

   

---
title: centos7 开启和关闭防火墙、端口命令
categories: [原创,笔记]
toc: true
date: 2020-01-09 10:03:09
tags: [linux, centos]
---
## 防火墙
* **查看防火墙状态**
> `firewall-cmd --state`
<!--more-->
* **关闭防火墙**
> `systemctl start firewalld.service`
* **禁止防火墙开机启动**
> `systemctl disable firewalld.service`
* **开启防火墙**
> `systemctl start firewalld.service`

## 端口
* **查看所有开启端口**
> `firewall-cmd --list-ports`
* **查看单个端口**
> `firewall-cmd --query-port=80/tcp`
* **永久开启端口**
> `firewall-cmd --zone=public --add-port=80/tcp --permanent`
* **关闭端口**
> `firewall-cmd --zone=public --remove-port=80/tcp --permanent`
* **重启防火墙**
> `firewall-cmd --reload`


## 参考链接
[**centos 7.3 开放端口并对外开放**](https://blog.csdn.net/qq_24232123/article/details/79781527)
[**Centos7,配置防火墙，开启端口**](https://blog.csdn.net/duzhanxiaosa/article/details/78890277)
[**CentOS7查看和关闭防火墙**](https://blog.csdn.net/ytangdigl/article/details/79796961)
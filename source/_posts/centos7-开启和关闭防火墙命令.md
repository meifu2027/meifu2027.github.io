---
title: centos7 开启和关闭防火墙命令
categories: [原创,笔记]
toc: true
date: 2020-01-09 10:03:09
tags: [linux, centos]
---
* **查看防火墙状态**
> `firewall-cmd --state`
<!--more-->
* **关闭防火墙**
> `systemctl start firewalld.service`
* **禁止防火墙开机启动**
> `systemctl disable firewalld.service`
* **开启防火墙**
> `systemctl start firewalld.service`
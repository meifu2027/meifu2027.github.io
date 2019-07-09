---
title: linux下常用中间件搭建 - activemq启动(八)
categories: [原创, 教程]
toc: true
date: 2019-07-09 17:03:00
tags: [linux, activemq]
---

## 概况
**activemq版本：** 5.12.0  
**activemq安装包：** apache-activemq-5.12.0.tar.gz 
<!--more-->
**其他依赖环境：**  
> jdk环境（前面章节已安装）


**服务器（1台）**  
> A：192.168.0.1  
> B：192.168.0.2   
> 此章节未做集群，任意选择一台安装即可。



## 步骤
### 解压


```bash
# 解压
tar -zxvf /u01/setup/apache-activemq-5.12.0.tar.gz -C /u01
# 启动
/u01/apache-activemq-5.12.0/bin/linux-x86-64/activemq start
# 停止
/u01/apache-activemq-5.12.0/bin/linux-x86-64/activemq stop
# 重启
/u01/apache-activemq-5.12.0/bin/linux-x86-64/activemq restart
# 日志查看
tail -200f  /u01/apache-activemq-5.12.0/data/activemq.log 
# 查看是否启动
ps -ef |grep activemq
ps aux |grep activemq

```



## 总结
不涉及使用说明，启动过于简单，没什么可说的。




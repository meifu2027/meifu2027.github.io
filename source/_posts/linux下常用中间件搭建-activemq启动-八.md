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

```
### 修改配置
```bash
vim /u01/apache-activemq-5.12.0/conf/activemq.xml
```

找到并修改成下面这段
```xml
<transportConnectors>
            <!-- DOS protection, limit concurrent connections to 1000 and frame size to 100MB -->
            <transportConnector name="openwire" uri="tcp://0.0.0.0:61616?maximumConnections=1000&amp;wireFormat.maxFrameSize=104857600&amp;wireFormat.maxInactivityDuration=0"/>
            <transportConnector name="amqp" uri="amqp://0.0.0.0:5672?maximumConnections=1000&amp;wireFormat.maxFrameSize=104857600&amp;wireFormat.maxInactivityDuration=0"/>
            <transportConnector name="stomp" uri="stomp://0.0.0.0:61613?maximumConnections=1000&amp;wireFormat.maxFrameSize=104857600&amp;wireFormat.maxInactivityDuration=0"/>
            <transportConnector name="mqtt" uri="mqtt://0.0.0.0:1883?maximumConnections=1000&amp;wireFormat.maxFrameSize=104857600&amp;wireFormat.maxInactivityDuration=0"/>
            <transportConnector name="ws" uri="ws://0.0.0.0:61614?maximumConnections=1000&amp;wireFormat.maxFrameSize=104857600&amp;wireFormat.maxInactivityDuration=0"/>
</transportConnectors>
```
### 常用命令
```bash
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

## 相关导航  
[*linux下常用中间件搭建(一)*](/2019/07/04/linux下常用中间件搭建一/)
[*linux下常用中间件搭建 - mysql搭建(二)*](/2019/07/04/linux下常用中间件搭建-mysql搭建二/)
[*linux下常用中间件搭建 - jdk安装(三)*](/2019/07/05/linux下常用中间件搭建-jdk安装三/)
[*linux下常用中间件搭建 - redis集群(四)*](/2019/07/05/linux下常用中间件搭建-redis集群四/)
[*linux下常用中间件搭建 - fastdfs集群(五)*](/2019/07/08/linux下常用中间件搭建-fastdfs集群五/)
[*linux下常用中间件搭建 - nginx搭建(六)*](/2019/07/08/linux下常用中间件搭建-nginx搭建-六/)
[*linux下常用中间件搭建 - zookeeper集群(七)*](/2019/07/09/linux下常用中间件搭建-zookeeper集群-七/)
[*linux下常用中间件搭建 - activemq启动(八)*](/2019/07/09/linux下常用中间件搭建-activemq启动-八/)
[*linux下常用中间件搭建 - maven安装(九)*](/2019/07/10/linux下常用中间件搭建-maven安装-九/)
[*linux下常用中间件搭建 - git安装(十)*](/2019/07/10/linux下常用中间件搭建-git安装-十/)
[*linux下常用中间件搭建 - oracle安装(十一)*](/2019/07/12/linux下常用中间件搭建-oracle安装-十一/)
[*linux下常用中间件搭建 - jboss部署(十二)*](/2019/08/30/linux下常用中间件搭建-JBoss部署-十二/)



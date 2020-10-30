---
title: Linux下自动切割Tomcat日志
categories:
  - 原创
toc: true
date: 2020-10-30 09:59:29
tags: [linux,logrotate]
---
各位在部署好应用之后，有没有日志文件太大的苦恼，这两天学了个新技能，使用logrotate命令来进行日志切割。
<!--more-->
## 配置logrotate切割配置文件
`vim /etc/logrotate.d/tomcat`
```bash
{ 
    copytruncate
    daily
    rotate 50
    missingok
    compress
    size 32M
}
```
配置信息参考自[*解决Tomcat日志文件catalina.out文件过大问题*](https://blog.51cto.com/lavenliu/1765791)

## 配置定时任务
正常是不需要配置定时任务，但是我这边不配置定时任务没有办法正常执行，原因不明，因此配置了定时任务。  
`vim /etc/crontab`
```bash
  0  1  *  *  * root logrotate -f /etc/logrotate.d/tomcat
```
执行完的结果如图  
![20200205-1.png](/img/blog/20201030-1.png)
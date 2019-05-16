---
title: linux下备份、替换、重启tomcat简单shell脚本
categories: [原创, 笔记]
toc: true
date: 2019-05-16 09:16:02
tags: [linux,shell,tomcat]
---

## 背景
最近在开发一个项目，部署应用的环境是linux、tomcat8，因为之前的项目都是由专业的实施人员负责部署，对于我来讲linux的掌握程度则相对较低。尽管在 [*Linux学习笔记 - （一）Linux达人养成计划 I --- 吐血整理*](/2018/04/11/Linux学习笔记-（一）Linux达人养成计划-I-吐血整理/)  中分享过自己学习的记录，但是并未真正实践过。
谨以此项目来练手第一个shell实战脚本。
<!--more-->
## 步骤
编写脚本之前首先要有明确要实现的功能和步骤。
**功能：** 备份原war、部署新war包。
**前提：** 新demo.war包已上传至/app/backup目录（这一步目前是手动操作）。
**脚本编写逻辑：**
1. 停应用；
2. 备份旧demo.war;
3. 替换新demo.war;
4. 重启tomcat;


## 脚本

```bash
#!/bin/bash
#以当天时间创建备份文件夹
BASEDATE=`date '+%Y%m%d'`
#先杀tomcat进程
ps -ef|grep tomcat|grep -v grep|awk '{print $2}'|xargs kill -9
#备份web包
backupdir=/app/backup/${BASEDATE}
if [ ! -d "${backupdir}" ]; then
    #若不存在当天文件夹，则新建一个
    mkdir -p ${backupdir}
fi
cd ${backupdir}
#若当天路径下还没有备份文件，则先进行备份
if [ ! -f demo.war ]; then
   cd /app/apache-tomcat-8.0.51/webapps
   echo "开始备份war包"
   cp /app/apache-tomcat-8.0.51/webapps/demo.war ${backupdir}/demo.war
   #判断备份命令的结果
   if [ $? -eq 0 ];
        then
        echo   " 备份完成"
        #删除原部署war包
        rm -f /app/apache-tomcat-8.0.51/webapps/demo.war
   else
        echo   " 备份失败"
   fi
else
    echo " 当天文件已备份，无需再次备份"
    #删除原部署war包
    rm -f /app/apache-tomcat-8.0.51/webapps/demo.war
fi
#将新上传war包移动到部署目录
mv /app/backup/demo.war /app/apache-tomcat-8.0.51/webapps/demo.war
#重启tomcat
cd /app/apache-tomcat-8.0.51/bin
sh startup.sh
```

## 总结
**由于是初次编写shell脚本，踩了一些坑，仅此记录：**
1. 在`fi` 结尾处多加了一个空格，导致语法报错，linux空格敏感；
2. 在windows上编辑了shell脚本，linux上报错`/bin/bash^M: 坏的解释器: 没有那个文件或目录`；  
原因是windows默认换行符为`\n\r`，而linux是`\n`，因此还需要在linux上执行替换的语句`sed -i 's/\r$//' demo.sh`

**另外，此脚本还不尽完善，只能在特定情况下执行才不会报错：**
* tomcat服务器处于运行状态；
* /app/backup 目录下有demo.war文件
* /app/apache-tomcat-8.0.51/webapps 目录下有demo.war文件

**希望以后能更多地掌握linux下的使用技巧，到时再来分享。**
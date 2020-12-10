---
title: Linux系统下Mysql定时任务备份脚本
categories:
  - 原创
toc: true
date: 2020-12-10 11:07:11
tags: [linux,mysql,shell]
---
生产系统需要在每天凌晨定时做数据库的备份，因此记录下数据库冷备脚本及配置过程。
<!--more-->
## 定时任务脚本
这里我们设置备份目录为`/u01/mysql/backup` (可设置任何自己的主要的目录作为备份目录)
开始写备份脚本 `vim /u01/mysql/backup/.job.sh`
```bash
#!/bin/sh
# 设置今天的日期
currenttime=$(date +%Y%m%d)
# 获取15天之前的日期
deltime=$(date -d "-15 day" +%Y%m%d)
# 定时备份数据库
mysqldump mydatabase >  /u01/mysql/backup/${currenttime}mydatabase.sql
# 定时删除日志信息
backcount=`find /u01/mysql/backup -mtime +15 -name *mydatabase.sql|wc -l`
if [ ${backcount} -gt 15 ];then
    find /u01/mysql/backup -mtime +15 -name *mydatabase.sql  -exec rm -f {} \;
fi
```
以上主要的备份逻辑为：
* 备份的数据库增加日期标识
* 判断是否有15个备份
* 若多于15个备份，则删除15天之前的数据
 
**注**：这种逻辑可能会有问题，如果15个备份都是在15天之前的，那么意味着所有备份都会被删除（由于我这里是每天一备份所以问题不大），还有种是根据文件名的排序删除最早的几个，这个就自由发挥了。  
写完执行脚本之后可以手动执行下脚本,查看备份目录下是否生成了新的备份`sh /u01/mysql/backup/.job.sh`


## 配置定时任务
定时任务的配置这里通过修改`/etc/crontab`文件来实现
```shell
vim /etc/crontab
# 增加下面一行，每天凌晨三点执行
  0  3  *  *  * root      sh /u01/mysql/backup/.job.sh
```

## 其他
若出现`mysqldump未找到命令`的报错，可能是mysql的bin目录并不在全局路径中，因此可以将上述目录配置到全局环境变量中`ln -s /u01/mysql/bin /usr/bin`，也可以针对具体命令进行软链接配置` ln -fs /u01/mysql/bin/mysqldump /usr/bin`
**注**：执行直接执行mysqldump命令还需要在my.cnf中有root用户密码的配置，详见[linux下常用中间件搭建 - mysql搭建(二)](/2019/07/04/linux下常用中间件搭建-mysql搭建二/)
## 相关链接
[linux下常用中间件搭建 - mysql搭建(二)](/2019/07/04/linux下常用中间件搭建-mysql搭建二/)
[Mysql数据库dump导出导入](/2019/04/28/Mysql数据库dump导出导入/)


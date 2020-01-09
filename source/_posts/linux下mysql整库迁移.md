---
title: linux下mysql整库迁移
categories: [原创, 教程]
toc: true
date: 2020-01-09 18:14:18
tags: [linux, mysql]
---
最近要进行服务器的整体迁移，此篇记录一下linux服务器下mysql数据库的整体迁移。mysql数据的搭建可查看之前的教程。 
<!--more--> 
[*linux下常用中间件搭建 - mysql搭建(二)*](/2019/07/04/linux下常用中间件搭建-mysql搭建二/)

## 整体迁移示例
**先看下整体迁移的图例，比较简单**
![mysql-migration.png](/img/blog/mysql-migration.png)

## 迁移步骤
1. **在新服务器B上面创建mysql用户**
```bash
groupadd mysql
useradd -g mysql -s /bin/false mysql
```
2. **在旧服务器A上面停止数据库服务**
```bash
service mysqld stop
```
3. **配置好免密的前提下开始同步**
```bash
# 在A服务器上执行
rsync -avzP /u01/mysql root@192.168.1.2:/u01
# 或在B服务器上执行
rsync -avzP root@192.168.1.1:/u01/mysql /u01
# rsync为增量同步指令，若下次有增量部分，可以重复执行2、3步骤即可
```
4. **拷贝旧服务器A上的配置文件到B相同目录**
```
/etc/my.cnf
/etc/init.d/mysqld
```
5. **新服务器B启动前配置**
```bash
# 迁移过来后可能会导致路径无法找到
# 方式一：增加软链接
ln -s  /u01/mysql /usr/local/mysql

# 方式二：修改mysqld_safe配置
vim /u01/mysql/bin/mysqld_safe
# 修改
MY_PWD='/usr/local/mysql'
# 改为
MY_PWD='/u01/mysql'
```
6. **新服务器B启动**
```bash
service mysqld start
```


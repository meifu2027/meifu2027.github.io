---
title: mysql下的dblink---FEDERATED引擎
categories: [原创, 笔记]
toc: true
date: 2020-02-11 13:58:24
tags: [mysql]
---
相信大部分使用过oracle的同学都知道DBLink这个东西，不同数据库连接的时候非常方便，那么mysql中是否也有呢？答案是肯定的。下面看看如何来配置吧。
<!--more-->
## 基本信息
> mysql版本：5.7.17
> 本地数据库L的ip: 192.168.0.1
> 远程数据库R的ip：192.168.0.2

## 启用federated引擎（两个数据库操作一样）
1. 使用`show engines;`命令查看数据库引擎开启情况，默认情况下`FEDERATED`的状态都是`NO`
2. 增加参数
```bash
vim /etc/my.cnf
# 在mysqld下增加配置
[mysqld]
federated
# 保存退出
```
3. 重启数据库`service mysqld restart`
4. 再次查看`show engines`应该`FEDERATED`的状态变成了`YES`
## 远程数据库建表
远程数据库正常建表即可
```sql
CREATE TABLE `book` (
  `book_id` int(11) NOT NULL,
  `book_name` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`book_id`)
)
```
## 本地数据库连接（方式一）
直接将连接信息配置在建表脚本中
```sql
-- username:password mysql用户名密码
-- 192.168.0.2:3306 远程数据库ip端口
-- database_name/book 数据库名/表名
CREATE TABLE remote_book (  
  `book_id` int(11) NOT NULL,
  `book_name` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`book_id`)
) ENGINE=FEDERATED CONNECTION='mysql://username:password@192.168.0.2:3306/database_name/book';
```
## 本地数据库连接（方式二）
直接在建表信息中包含远程数据库的账号密码安全性比较低
因此还有另外一种方式
```sql
-- 创建全局连接（需要root用户执行，普通用户没有权限）
CREATE SERVER remote_database
FOREIGN DATA WRAPPER mysql
OPTIONS (USER 'username', PASSWORD 'password', HOST '192.168.0.2', PORT 3306, DATABASE 'database_name');
-- 建表
CREATE TABLE remote_book (  
  `book_id` int(11) NOT NULL,
  `book_name` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`book_id`)
) ENGINE=FEDERATED CONNECTION='remote_database/book'
```
## 测试连接
```sql
-- 在本地数据库查看自己建的表，这个表的内容实际上是存在远程数据库的
-- 增删改查的权限取决于远程数据库分配的用户的权限
select * from remote_book;
```

## 相关链接
[**用mysql实现类似于oracle dblink的功能**](https://blog.csdn.net/langkeziju/article/details/50462943)
[**MySQL下的DB Link**](https://juejin.im/post/5d1d75dcf265da1ba4320ab3#heading-0)
---
title: oracle使用sqlldr快速导入txt数据（实测20秒90w条）
categories:
  - 原创
toc: true
date: 2021-07-20 11:58:37
tags: [linux, oracle]
---
> 无论是新系统上线还是用数据库处理数据，当需要有大数据量的导入的时候，你会采取哪种方式呢？  
今天记录下linux下oracle数据库使用`sqlldr`命令快速导入数据的操作。

<!-- more -->
## txt生成
一般我们将数据整理到excel中之后转存为txt文档。
![保存为txt](/img/blog/20210720-1-保存为txt.jpg)
然后将生产的txt文件上传至数据库服务器
##  执行脚本
切换至oracle用户`su - oracle`
### ctl脚本
```bash
# skip=1表示跳过第1行数据，skip=0表示不跳过数据
OPTIONS (skip=1) 
load data
# 不指定编码可能会存在导入中文乱码的情况，编码可以通过sql查询`select VALUE from nls_database_parameters where parameter ='NLS_CHARACTERSET';`
CHARACTERSET ZHS16GBK
# 需要导入的txt文件
INFILE  '/u01/20210720-1.txt'  
# 指定数据存入目标表
append into  table DEMO_TABLE
# X'09'为为制表符 ，如果指定其他分隔符可以替换，如'|'或','等都可以
FIELDS TERMINATED BY X'09'
# 输入要导入的字段，需要与txt上对应起来
trailing nullcols
(
column1,
column2,
column3,
column4,
column5,
column6,
column7,
column8,
column9
)

```
### 执行命令
```bash
# oracle的账号密码输入 指定要执行的ctl脚本  指定日志文件路径
sqlldr account/password control =/u01/load.ctl log=load.log;
```

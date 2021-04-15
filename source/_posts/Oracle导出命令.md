---
title: Oracle导出命令exp
categories:
  - 原创
toc: true
date: 2021-04-15 11:29:35
tags: [linux, oracle]
---
## 表导出
```bash
exp username/password@sidname file=/tmp/filename.dmp tables=table1,table2
```
<!-- more -->
## 库导出
```bash
exp username/password@/sidname file=/tmp/filename.dmp owner=username
```
## 监听配置
linux监听目录：`{$ORACLE_HOME}/product/12.1.0/dbhome_1/network/admin/tnsnames.ora`（不配置导出会报错）
```
-- 样例
sidname =
  (DESCRIPTION =
    (ADDRESS_LIST =
      (ADDRESS = (PROTOCOL = TCP)(HOST = 192.168.0.1)(PORT = 1521))
    )
    (CONNECT_DATA =
      (SID = sidname)
      (SERVER = DEDICATED)
    )
  )
```

## 参考
[*Oracle expdp导出多表或表中的部分数据*](http://blog.itpub.net/16582684/viewspace-755072/)
[*Oracle中用exp/imp命令快速导入导出数据库或者导出指定用户下的部分表或数据*](https://blog.csdn.net/qq_36135335/article/details/81704034)

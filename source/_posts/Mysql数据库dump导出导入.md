---
title: Mysql数据库dump导出导入
toc: true
date: 2019-04-28 12:44:01
tags: mysql
categories: [原创, 笔记]
---
记录mysql数据库导出导入命令
<!--more-->
```bash
# 全量导出脚本
mysqldump source_database  -uroot -p > exportName.sql
# 全量导入脚本
mysql target_database -uroot -p < exportName.sql

# 单表导出
mysqldump source_database tableName -uroot -p > exportName.sql
# 全量导入脚本
mysql target_database tableName -uroot -p < exportName.sql

```
---
title: Oracle下使表（table）对应的索引（index）和约束（constraint）生效和失效的操作
categories:
  - 原创
toc: true
date: 2021-04-22 14:49:36
tags: [linux, oracle, 索引, 约束]
---
## 背景
> 最近做Oracle的表同步，发现全量插入的时候，目标表的索引和主键的存在会对插入速度的影响很大（千万级表尤为明显），特此记录。

<!-- more -->
## 约束生效失效
```sql
-- 约束失效 （约束包含主键约束、非空约束、外键约束等）
-- TABLE_NAME 表名
-- CONSTRAINT_NAME 约束名称
ALTER TABLE TABLE_NAME  DISABLE CONSTRAINT CONSTRAINT_NAME;
-- 约束生效
ALTER TABLE TABLE_NAME  ENABLE CONSTRAINT CONSTRAINT_NAME;

```
## 索引生效失效
```sql
-- 索引失效 
-- INDEX_NAME 索引名称
ALTER INDEX INDEX_NAME UNUSABLE;
-- 索引生效
ALTER INDEX INDEX_NAME REBUILD;

```

## 根据表名查询约束、索引
这里为了方便使用，直接把根据表名查询生效、失效脚本列出来
**约束查询脚本**
```sql
-- 'TABLE_NAME' 为需要查询的表名
SELECT TABLE_NAME,
       'ALTER TABLE ' || TABLE_NAME || ' DISABLE CONSTRAINT ;' ||
       CONSTRAINT_NAME 约束失效脚本,
       'ALTER TABLE ' || TABLE_NAME || ' ENABLE CONSTRAINT ;' ||
       CONSTRAINT_NAME 约束生效脚本
  FROM USER_CONS_COLUMNS
 WHERE TABLE_NAME = 'TABLE_NAME'
   AND POSITION IS NOT NULL -- 这个条件用于查询主键约束，非空约束该字段为空
 ORDER BY TABLE_NAME;
```
**索引查询脚本**
```sql
-- 'TABLE_NAME' 为需要查询的表名
-- 由于约束也会有创建索引（比如主键约束），因此排除跟约束绑定的索引
SELECT T.TABLE_NAME,
       T.STATUS 索引生效失效标志位,
       'ALTER INDEX ' || T.INDEX_NAME || ' UNUSABLE;' 索引失效脚本,
       'ALTER INDEX ' || T.INDEX_NAME || ' REBUILD;' 索引生效脚本
  FROM USER_INDEXES T
 WHERE T.TABLE_NAME = 'TABLE_NAME'
   AND T.INDEX_NAME NOT IN
       (SELECT P.CONSTRAINT_NAME
          FROM USER_CONS_COLUMNS P
         WHERE P.TABLE_NAME = 'TABLE_NAME'
           AND P.POSITION IS NOT NULL)
 ORDER BY T.TABLE_NAME;
```

## 索引、约束删除脚本

```sql
-- 约束删除
alter table TABLE_NAME drop constraint CONSTRAINT_NAME;
-- 索引删除
drop INDEX INDEX_NAME;

```

## 特别注意

本次在同步验证的时候，发现如果`truncate table` 命令如果在使索引生效之后，那么失效的索引又会生效。因此一定要保证先执行`truncate table`命令，再执行`ALTER TABLE TABLE_NAME  DISABLE CONSTRAINT CONSTRAINT_NAME`命令。经验证，对`constraint`没有影响。

---
title: mysql开启定时任务
categories: [转载, 笔记]
toc: true
date: 2020-02-18 14:29:01
tags: [mysql]
---
需要用定时任务处理报表分析数据，记录下mysql定时任务开启。
<!--more-->
## 定时任务创建测试
```sql
-- 1、创建测试表
DROP TABLE IF EXISTS test;

CREATE TABLE test (
	id INT (11) NOT NULL auto_increment PRIMARY KEY,
	time datetime NOT NULL
) ENGINE = INNODB DEFAULT charset = utf8;

-- 2、创建测试存储过程
DROP PROCEDURE IF EXISTS test_proce ;

CREATE PROCEDURE test_proce ()
BEGIN
	INSERT INTO test (time) VALUES (now());
END

-- 3、开启event
SHOW VARIABLES LIKE 'event_scheduler';
-- 若`event_scheduler`状态是OFF 则设置为ON，可能需要root用户执行
SET GLOBAL event_scheduler = 'on';

-- 4、创建事件（每1秒钟插入一条数据）
DROP EVENT IF EXISTS test_event;

CREATE EVENT test_event ON SCHEDULE EVERY 1 SECOND ON COMPLETION PRESERVE DISABLE DO
	CALL test_proce ();

-- 5、开启事件执行定时任务
ALTER EVENT test_event ON COMPLETION PRESERVE ENABLE;

-- 6、关闭定时任务
ALTER EVENT test_event ON COMPLETION PRESERVE DISABLE;

-- 7、查看测试表
SELECT * FROM test;
```

## event时间设置规则
```sql
-- EVERY 后面的是时间间隔，可以选 1 second，3 minute，5 hour，9 day，1 month，1 quarter（季度），1 year 
CREATE EVENT test_event ON SCHEDULE EVERY 1 DAY STARTS '2012-09-24 00:00:00' ON COMPLETION PRESERVE ENABLE DO
	CALL test_procedure ();

-- 从2013年1月13号0点开始，每天运行一次
ON SCHEDULE EVERY 1 DAY STARTS '2013-01-13 00:00:00';

-- 从现在开始每隔九天定时执行
ON SCHEDULE EVERY 9 DAY STARTS NOW() ;

-- 每个月的一号凌晨1 点执行
ON SCHEDULE EVERY 1 MONTH STARTS date_add(
	date_add(
		date_sub(
			curdate(),
			INTERVAL DAY (curdate()) - 1 DAY
		),
		INTERVAL 1 MONTH
	),
	INTERVAL 1 HOUR
);

-- 每个季度一号的凌晨1点执行
ON SCHEDULE EVERY 1 QUARTER STARTS date_add(
	date_add(
		date(
			concat(
				YEAR (curdate()),'-',elt(QUARTER (curdate()),1,4,7,10),'-',1
			)
		),
		INTERVAL 1 QUARTER
	),
	INTERVAL 1 HOUR
);
```
## 参考链接
[**Mysql 查看定时器 打开定时器 设置定时器时间**](https://blog.csdn.net/qq_34082034/article/details/56290920)
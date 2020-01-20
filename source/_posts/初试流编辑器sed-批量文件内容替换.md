---
title: 初试流编辑器sed-批量文件内容替换
categories: [原创]
toc: true
date: 2020-01-20 14:13:45
tags: [sed]
---
迁移环境的时候需要批量修改配置文件中的ip地址，初试下流编辑器sed的强大。
<!--more-->
```bash
# 将找到的所有符合条件的文件中的ip 192.168.1.1 替换成 192.168.1.2
# 注：sed后需要加上 -i 参数，否则只是把替换的结果打印出来而已
find /u01/jboss -name standalone.conf |xargs sed -i 's/192.168.1.1/192.168.1.2/g'
```
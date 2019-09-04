---
title: linux下常用中间件搭建 - mysql搭建(二)
categories: [原创, 教程]
toc: true
date: 2019-07-04 20:22:48
tags: [linux,mysql]
---



## 概况
**mysql版本：** 5.7.17  
**mysql安装包：** mysql-5.7.17-linux-glibc2.5-x86_64.tar.gz
<!--more-->
## 步骤
### 解压、新建目录


```bash
# 解压到/u01目录下
tar -zxvf /u01/setup/mysql-5.7.17-linux-glibc2.5-x86_64.tar.gz -C /u01
# 解压完后重命名为mysql
mv /u01/mysql-5.7.17-linux-glibc2.5-x86_64 /u01/mysql
# 新建data、log目录，后面my.cnf配置会用到
mkdir /u01/mysql/data  /u01/mysql/log

```
### 新建用户组、用户、授权

```bash
# 添加用户组
groupadd mysql
# 创建不允许登录用户
useradd -g mysql -r -s /sbin/nologin -M  mysql
# 目录授权
chown -R mysql:mysql /u01/mysql
```

### mysql基本配置

```bash
# 拷贝启动服务
cp /u01/mysql/support-files/mysql.server /etc/init.d/mysqld
# 修改配置并保存（如图1）
vim /etc/init.d/mysqld
# 执行这步
ldconfig
# 设置环境变量
echo "PATH=$PATH:/u01/mysql/bin" > /etc/profile.d/mysql.sh
# 使环境变量生效
source /etc/profile.d/mysql.sh
# 设置开机启动
chkconfig mysqld on
# 配置my.cnf 如图2（默认配置）图3（修改后配置）可根据实际需要修改
vim /etc/my.cnf
```


```bash
[mysqld]
server-id=1
skip-name-resolve
max_connections=10000
lower_case_table_names=1
basedir=/u01/mysql
datadir=/u01/mysql/data
socket=/u01/mysql/data/mysql.sock
log-error=/u01/mysql/log/mysqld.log
sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES
port=3306
explicit_defaults_for_timestamp=true
max_allowed_packet=500M
innodb_buffer_pool_size=8G
expire_logs_days=7
log_bin=binlog
binlog_format=row
binlog-ignore-db=oggddl
wait_timeout=2880000
interactive_timeout=2880000

[client]
socket=/u01/mysql/data/mysql.sock

[mysqld_safe]
log-error=/u01/mysql/log/mysqld.log
pid-file=/u01/mysqld/mysqld.pid
```

```bash
# 创建软链接(否则会报错：mysqld_safe The file /usr/local/mysql/bin/mysqld does not exist or is not executable.)
mkdir -p /usr/local/mysql/bin
ln -s /u01/mysql/bin/mysqld /usr/local/mysql/bin/mysqld  

# 执行初始化 --initialize-insecure 生成空密码  --initialize 生成随机密码
/u01/mysql/bin/mysqld --initialize-insecure --user=mysql --basedir=/u01/mysql --datadir=/u01/mysql/data
```

图1（配置修改）
![1.png](https://i.loli.net/2019/07/04/5d1dfa1fdbd5698153.png)
图2（my.cnf原始配置)
![2.png](https://i.loli.net/2019/07/04/5d1dfcd24cfc384799.png)
图3（my.cnf修改后配置)
![3.png](https://i.loli.net/2019/07/04/5d1e04303f70047098.png)

### 后续配置


```bash
# 数据库启动
service mysqld start
# 数据库停止
service mysqld stop
# 数据库重启
service mysqld restart

# 启动数据库后 进入mysql命令行
mysql -u root
# 修改初始密码（此时Navicat还不能直接链接）
update mysql.user set authentication_string=password('root') where user='root';
# 配置远程连接
grant all privileges on *.* to 'root'@'%' identified by 'root';
# 刷新权限(然后就可以连接测试啦)
flush privileges;
# 退出mysql命令行
exit
```
**为了执行有些步骤无需输入用户密码，也可以直接配到my.cnf中去**
```bash
# 修改配置
vim /etc/my.cnf
# 增加内容
[client]
socket=/u01/mysql/data/mysql.sock
host=localhost
user=root
password=root
port=3306

[mysqladmin]
user=root
password=root

[mysqldump]
user=root
password=root

[mysqlshow]
user=root
password=root

# 保存后重启数据库
service mysqld restart
```

## 创建数据库
生产上使用一般不会直接使用root用户，往往会创建一个业务数据库和操作这个数据库的用户。
```sql
-- 1.打开Navicat用root用户登录
-- 2.打开mysql数据库查询页面
-- 3.更新root授权权限
update user t set t.Grant_priv = 'Y' WHERE t.`Host`='%' and t.`User` = 'root';
-- 4.创建数据库
create database demodatabase;
-- 5.创建用户并授权
CREATE USER 'demouser'@'%' IDENTIFIED BY '123456';
GRANT SELECT, INSERT, UPDATE, REFERENCES, DELETE, CREATE, DROP, ALTER, INDEX, TRIGGER, CREATE VIEW, SHOW VIEW, EXECUTE, ALTER ROUTINE, CREATE ROUTINE, CREATE TEMPORARY TABLES, LOCK TABLES, EVENT ON `demodatabase`.* TO 'demouser'@'%';
-- 6. 切换到demouser/123456登录
```
**初始数据库的导入请参见**[Mysql数据库dump导出导入](/2019/04/28/Mysql数据库dump导出导入/)

## 总结
*发现写文档比搭建一遍更累。*  
*踩坑难免的，比如：*  
- *要是`/u01/mysql/data`目录和`/u01/mysql/log`目录要是在* **目录授权** *后才建的，那么会导致初始化的时候可能会没有写权限 ；*
## 相关导航  
[*linux下常用中间件搭建(一)*](/2019/07/04/linux下常用中间件搭建一/)
[*linux下常用中间件搭建 - mysql搭建(二)*](/2019/07/04/linux下常用中间件搭建-mysql搭建二/)
[*linux下常用中间件搭建 - jdk安装(三)*](/2019/07/05/linux下常用中间件搭建-jdk安装三/)
[*linux下常用中间件搭建 - redis集群(四)*](/2019/07/05/linux下常用中间件搭建-redis集群四/)
[*linux下常用中间件搭建 - fastdfs集群(五)*](/2019/07/08/linux下常用中间件搭建-fastdfs集群五/)
[*linux下常用中间件搭建 - nginx搭建(六)*](/2019/07/08/linux下常用中间件搭建-nginx搭建-六/)
[*linux下常用中间件搭建 - zookeeper集群(七)*](/2019/07/09/linux下常用中间件搭建-zookeeper集群-七/)
[*linux下常用中间件搭建 - activemq启动(八)*](/2019/07/09/linux下常用中间件搭建-activemq启动-八/)
[*linux下常用中间件搭建 - maven安装(九)*](/2019/07/10/linux下常用中间件搭建-maven安装-九/)
[*linux下常用中间件搭建 - git安装(十)*](/2019/07/10/linux下常用中间件搭建-git安装-十/)
[*linux下常用中间件搭建 - oracle安装(十一)*](/2019/07/12/linux下常用中间件搭建-oracle安装-十一/)
[*linux下常用中间件搭建 - jboss部署(十二)*](/2019/08/30/linux下常用中间件搭建-JBoss部署-十二/)

----
*其他的就先不写了，就先这样吧，后面还有要改的再补充；*
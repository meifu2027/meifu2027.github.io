---
title: linux下oracle整库迁移
categories: [原创, 教程]
toc: true
date: 2020-01-19 16:47:51
tags: [linux, oracle]
---
最近要进行服务器的整体迁移，此篇记录一下linux服务器下oracle数据库的整体迁移。mysql数据的搭建可查看之前的教程。 
<!--more--> 
[*linux下常用中间件搭建 - oracle安装(十一)*](/2019/07/12/linux下常用中间件搭建-oracle安装-十一/)

## 整体迁移示例
**先看下整体迁移的图例，比较简单**
![oracle-migration.png](/img/blog/oracle-migration.png)

## 迁移步骤
1. **在新服务器B上面创建oracle用户**
```bash
# 建立用户和组
groupadd oinstall  
groupadd dba  
groupadd oper  
useradd -g oinstall -G dba,oper oracle  
# oracle用户的登录密码，后续登录要用，记着。
echo "123456" | passwd --stdin oracle
```
2. **修改基本配置**
```bash
# 三个文件如下，具体配置可参见oracle安装
/etc/sysctl.conf
/etc/security/limits.conf
/etc/pam.d/login
```
3. **在旧服务器A上面停止数据库服务**
```bash
# 切换oracle用户
su - oracle
# 进入sql命令行
sqlplus / as sysdba
# 停止服务
shutdown immediate
# 退出命令行
exit
# 停止监听
lsnrctl stop
```

4. **配置好免密的前提下开始同步**
```bash
# 在A服务器上执行
rsync -avzP /u01/orcl root@192.168.0.2:/u01
# 或在B服务器上执行
rsync -avzP root@192.168.0.1:/u01/orcl /u01
# rsync为增量同步指令，若下次有增量部分，可以重复执行2、3步骤即可
```
5. **拷贝旧服务器A上的配置到B相同文件**
```bash
# 切换到oracle用户
su - oracle
# 修改环境变量
vim ~/.bash_profile
# 文件最后添加
ORACLE_SID=orcl
ORACLE_BASE=/u01/orcl/oracle
ORACLE_HOME=$ORACLE_BASE/product/12.2.0/db_1
LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$ORACLE_HOME/lib
SQL_PATH=/home/oracle/sql
PATH=$PATH:$HOME/bin:$ORACLE_HOME/bin
export PATH ORACLE_BASE ORACLE_HOME ORACLE_SID LD_LIBRARY_PATH SQL_PATH NLS_LANG

# 生效
source ~/.bash_profile
```
6. **修改监听相关配置**
```bash
vim /u01/orcl/oracle/product/12.2.0/db_1/network/admin/listener.ora
vim /u01/orcl/oracle/product/12.2.0/db_1/network/admin/tnsnames.ora
# 修改其中的HOST 为新服务器ip 192.168.0.2

# 若存在与mysql的dblink，需要同步另外两个文件
/etc/odbc.ini
/etc/odbcinst.ini
```

7. **新服务器B启动**
```bash
# 切换oracle用户
su - oracle
# 进入sql命令行
sqlplus / as sysdba
# 启动服务
startup
# 退出命令行
exit
# 启动监听
lsnrctl start

```

8. **若存在mysql的dblink**
```bash
# 安装unixODBC
yum install unixODBC 
# 拷贝MySQLodbc的lib包到系统lib目录（也可另建目录并在配置文件中指定）
cp /u01/setup/mysql-connector-odbc-5.3.8-linux-glibc2.12-x86-64bit/lib/libmyodbc5w.so /usr/lib64/
# 执行命令   验证odbc配置是否有问题
isql -v myodbcct
# 最后PL/SQL登录后进行查询验证数据同步情况、dblink等
```

## 相关导航  
[*linux下mysql整库迁移*](/2020/01/09/linux下mysql整库迁移/)
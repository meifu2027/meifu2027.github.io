---
title: linux下常用中间件搭建 - oracle安装(十一)
categories: [原创, 教程]
toc: true
date: 2019-07-12 19:47:20
tags: [linux, oracle]
---

## 概况
**oracle版本：** 12.2.0.1.0  
**oracle安装包：** linuxx64_12201_database.zip(官网下的两个包合到一起了) 
<!--more-->
**其他依赖环境：**  
> xmanager_6.zip （图形化界面安装oracle使用）  
> compat-libcap1-1.10-1.x86_64.rpm  
> compat-libstdc++-33-3.2.3-61.x86_64.rpm  
> ksh-20120801-37.el6_9.x86_64.rpm  
> libaio-devel-0.3.107-10.el6.x86_64.rpm  

说明：其实oracle的安装需要依赖很多包，我这里列出来的是我在安装过程中缺少的包。具体可以查阅详细的依赖。如果是联网的环境可以先安装：

```bash
yum -y install binutils compat-libcap1 compat-libstdc++-33 compat-libstdc++-33*.i686 elfutils-libelf-devel gcc gcc-c++ glibc*.i686 glibc glibc-devel glibc-devel*.i686 ksh libgcc*.i686 libgcc libstdc++ libstdc++*.i686 libstdc++-devel libstdc++-devel*.i686 libaio libaio*.i686 libaio-devel libaio-devel*.i686 make sysstat unixODBC unixODBC*.i686 unixODBC-devel unixODBC-devel*.i686 libXp

```
**服务器（1台）**  
> A：192.168.0.1  
> B：192.168.0.2   
> 安装任意一台。
## 基本配置

### 新建用户组、用户
```bash
# 新建目录
mkdir -p /u01/orcl/oracle/product/12.2.0/db_1
# 解压oracle目录
unzip /u01/setup/linuxx64_12201_database.zip -d /u01/orcl/oracle

# 建立用户和组
groupadd oinstall  
groupadd dba  
groupadd oper  
useradd -g oinstall -G dba,oper oracle  
# oracle用户的登录密码，后续登录要用，记着。
echo "123456" | passwd --stdin oracle
# 授权目录
chown -R oracle:oinstall /u01/orcl 
chmod -R 775 /u01/orcl
```

### 修改内核参数
```bash
# 修改内核参数
vim /etc/sysctl.conf

# 文件改成
kernel.sysrq = 0
kernel.core_uses_pid = 1
kernel.msgmnb = 65536
kernel.msgmax = 65536
kernel.shmall = 4294967296
kernel.shmmax = 68719476736
kernel.shmmni = 4096
kernel.sem = 250 32000 100 128

fs.aio-max-nr = 1048576
fs.file-max = 6815744

net.ipv4.ip_forward = 0
net.ipv4.ip_local_port_range = 9000 65500
net.ipv4.tcp_syncookies = 1
net.ipv4.conf.default.rp_filter = 1
net.ipv4.conf.default.accept_source_route = 0
net.core.rmem_default = 262144
net.core.rmem_max = 4194304
net.core.wmem_default = 262144
net.core.wmem_max = 1048576

# 使修改生效
sysctl -p
```

### 修改文件限制
```bash
# 修改文件限制
vim /etc/security/limits.conf
# 最后添加修改内容
oracle         soft    nofile         131072
oracle         hard    nofile         131072
oracle         soft    nproc          131072
oracle         hard    nproc          131072
oracle         soft    core           unlimited
oracle         hard    core           unlimited
oracle         soft    memlock        unlimited
oracle         hard    memlock        unlimited

# 修改配置
vim /etc/pam.d/login
# 最后添加修改内容
session    required     /lib64/security/pam_limits.so
```

### 修改oracle用户环境变量

```bash
vim ~oracle/.bash_profile
# 最后添加
ORACLE_SID=orcl
ORACLE_BASE=/u01/orcl/oracle
ORACLE_HOME=$ORACLE_BASE/product/12.2.0/db_1
LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$ORACLE_HOME/lib
SQL_PATH=/home/oracle/sql
PATH=$PATH:$HOME/bin:$ORACLE_HOME/bin
export PATH ORACLE_BASE ORACLE_HOME ORACLE_SID LD_LIBRARY_PATH SQL_PATH NLS_LANG

# 修改好使生效
source ~oracle/.bash_profile
```

## oracle用户开始安装
```bash
# 切换用户
su oracle 
# 检查配置
xhost +
# 如果出现以下信息
xhost: command not found
# 检查一下需要安装什么依赖包
yum whatprovides "*/xhost"
# 我这里显示如图1，安装下rpm离线包（我这里yum可以直接安装 ，若不支持可以下载下来安装）
# 如果执行不了可能要切换到root用户执行
yum install xorg-x11-server-utils-7.5-13.el6.x86_64
# 还要再安装一下xdpyinfo，否则后面可能出现报错：>>> Could not execute auto check for display colors using command /usr/bin/xdpyinfo. Check if the DISPLAY variable is set.    Failed <<<<
# 同样的，我这里的yum源可以下得到我就直接下了，否则就下载rpm包然后安装
yum install -y xdpyinfo
```
图1  
![6.png](https://i.loli.net/2019/07/12/5d28442b6459657095.png)
### 先准备好Xmanager
[教程你懂得](https://www.aiweibk.com/17887.html)  
**前提是服务器已经装了**`xterm`，**没有的话要先装一下**`yum install -y xterm`  
**一切准备就绪后打开xstart，配置如下连接：  **  
`/usr/bin/xterm -ls -display $DISPLAY`  
![4.png](https://i.loli.net/2019/07/12/5d28419308b2991161.png)
**如果没有安装过**`yum install –y xorg-x11-xauth` **连接可能出现如下报错：**
![5.png](https://i.loli.net/2019/07/12/5d2844f1b030b52690.png)
**连接成功之后会出现如下图**  
```bash
cd /u01/orcl/oracle/database
# 不设置会出现中文乱码
export LANG=en_us
./runInstaller
```
**好啦，终于吧这界面请出来了，开始愉快的可视化界面安装吧**  
![7.png](https://i.loli.net/2019/07/12/5d284aea6058c51945.png)
## oracle可视化界面安装
1.
![oracle图形界面步骤1.png](https://i.loli.net/2019/07/12/5d285ef2a399f33490.png)
2.
![oracle图形界面步骤2.png](https://i.loli.net/2019/07/12/5d285ef2b57f849443.png)
3.
![oracle图形界面步骤3.png](https://i.loli.net/2019/07/12/5d285ef2ce40349636.png)
4.
![oracle图形界面步骤4.png](https://i.loli.net/2019/07/12/5d285ef30e25b18949.png)
5.
![oracle图形界面步骤5.png](https://i.loli.net/2019/07/12/5d285ef3585cd59812.png)
6.
![oracle图形界面步骤6.png](https://i.loli.net/2019/07/12/5d285ef346aca68908.png)
7.
![oracle图形界面步骤7.png](https://i.loli.net/2019/07/12/5d285ef33865241412.png)
8.
![oracle图形界面步骤8-1.png](https://i.loli.net/2019/07/12/5d285ef32c24a23319.png)
9.
![oracle图形界面步骤9.png](https://i.loli.net/2019/07/12/5d285ef279a8626273.png)
10.
![oracle图形界面步骤10.png](https://i.loli.net/2019/07/12/5d285f0c5c15514507.png)
11.
![oracle图形界面步骤11-1.png](https://i.loli.net/2019/07/12/5d285f0c7076b75077.png)
12.
![oracle图形界面步骤12-1.png](https://i.loli.net/2019/07/12/5d285f0c895aa61955.png)
13.
![oracle图形界面步骤13.png](https://i.loli.net/2019/07/12/5d285f0c94b2390885.png)
14.
![oracle图形界面步骤14.png](https://i.loli.net/2019/07/12/5d285f0c9b84d85360.png)
15.
![oracle图形界面步骤15.png](https://i.loli.net/2019/07/12/5d285f0ca3e4032449.png)
16.
![oracle图形界面步骤16.png](https://i.loli.net/2019/07/12/5d285f0cb52eb32963.png)
17.
![oracle图形界面步骤17-1.png](https://i.loli.net/2019/07/12/5d285f2a557a612861.png)
![oracle图形界面步骤17-2.png](https://i.loli.net/2019/07/12/5d285f2aa520e98737.png)
18.
![oracle图形界面步骤18.png](https://i.loli.net/2019/07/12/5d285f2a0005146899.png)
19.
![oracle图形界面步骤19-1.png](https://i.loli.net/2019/07/12/5d285f2a1df6460391.png)
```bash
# 增加交换空间（oracle会根据系统的内存大小设置swap size的下限，我的服务器是32G内存，所以最小是16G）

# 查看系统RAM大小
free -m
# 创建swap
dd if=/dev/zero of=/u01/swap bs=1M count=16384
# 格式化分区文件
mkswap /u01/swap
# 激活swap，立即启用交换分区文件
swapon /u01/swap
# 如果需要关闭（不执行）
swapoff /u01/swap
#再次 查看系统RAM大小
free -m
# 为了使操作系统在重启后swap自动挂载，要修改/etc/fstab文件
vim /etc/fstab
# 最后加上
/u01/swap swap swap defaults 0 0

```

```bash
# 解决包依赖
cd /u01/setup
rpm -ivh compat-libcap1-1.10-1.x86_64.rpm compat-libstdc++-33-3.2.3-61.x86_64.rpm ksh-20120801-37.el6_9.x86_64.rpm libaio-devel-0.3.107-10.el6.x86_64.rpm
```
配置完成后再`check again`
20.
![oracle图形界面步骤20.png](https://i.loli.net/2019/07/12/5d285f2a2d30236406.png)
21.
![oracle图形界面步骤21.png](https://i.loli.net/2019/07/12/5d285fe1c09c598157.png)
22.
![oracle图形界面步骤22.png](https://i.loli.net/2019/07/12/5d285f2a88ce512543.png)
23.
![oracle图形界面步骤23.png](https://i.loli.net/2019/07/12/5d2862fa81ee072299.png)
24.
把主机名替换成ip访问，用之前配置的账号密码登录，如：sys/123456
![oracle图形界面步骤24.png](https://i.loli.net/2019/07/12/5d285f2a94d8288170.png)
25.
![oracle图形界面步骤25.png](https://i.loli.net/2019/07/12/5d28640f97f0542831.png)
## 后续配置
### 服务端配置
后续的配置可以不需要xstart了，直接用oracle用户登上去配置。
```bash
# 配置环境变量
vim ~/.bashrc
# 最后添加
export ORACLE_BASE=/u01/orcl/oracle
export ORACLE_HOME=/u01/orcl/oracle/product/12.2.0/db_1
export PATH=$PATH:$ORACLE_HOME/bin
export ORACLE_SID=orcl
# 保存后使生效
source ~/.bashrc

# 编辑配置使客户端可以连接
vim /u01/orcl/oracle/product/12.2.0/db_1/network/admin/sqlnet.ora
# 新增
SQLNET.ALLOWED_LOGON_VERSION_SERVER=8
SQLNET.ALLOWED_LOGON_VERSION_CLIENT=8
SQLNET.ALLOWED_LOGON_VERSION=8
```
### 客户端配置
我是使用plsql\development作为客户端连接工具，并且你的本地需要先安装oracle客户端（此步骤省略）。
**配置客户端**
一般会在 `%ORACLE_CLIENT_HOME%\Oracle\NETWORK\ADMIN`目录下有个`tnsnames.ora`，最后添加内容
```bash
demo =
  (DESCRIPTION =
    (ADDRESS_LIST =
      (ADDRESS = (PROTOCOL = TCP)(HOST = 192.168.0.1)(PORT = 1521))
    )
    (CONNECT_DATA =
      (SID = orcl)
      (SERVER = DEDICATED)
    )
  )
```
配置好之后可以登录  
![9.png](https://i.loli.net/2019/07/12/5d2871da4177e84455.png)
由于我这里还有一个比较奇怪的事情是：**如果我不修改密码，那么无论如何客户端都会报账号密码错误，因此又把**`sys`、`system`**两个用户的账号密码修改了一下。这个还是再服务器上以oracle用户修改：**
```bash
sqlplus / as sysdba
alter user system identified by 123456;
alter user sys identified by 123456;
```
再次登录，如下图。这样就初步完成了oracle的安装   
![8.png](https://i.loli.net/2019/07/12/5d28717c0eb1d24535.png)

## 导数
### 创建表空间

```sql
-- 管理员用户创建表空间
CREATE SMALLFILE 
    TABLESPACE "DEMO_TBS" 
    LOGGING 
    DATAFILE '/u01/orcl/oracle/oradata/orcl/DEMO001.dbf' SIZE 1024M AUTOEXTEND 
    ON NEXT  8192K MAXSIZE UNLIMITED, '/u01/orcl/oracle/oradata/orcl/DEMO002.dbf' SIZE 1024M AUTOEXTEND 
    ON NEXT  8192K MAXSIZE UNLIMITED EXTENT MANAGEMENT LOCAL 
    SEGMENT SPACE MANAGEMENT  AUTO ;
```
### 创建用户

```sql
-- Create the user 
create user demouser
  identified by "123456"
  default tablespace DEMO_TBS
  temporary tablespace TEMP
  profile DEFAULT ;
-- Grant/Revoke role privileges 
grant connect to demouser;
grant exp_full_database to demouser;
grant imp_full_database to demouser;
grant resource to demouser;

-- 设置用户表空间配额
alter user demouser quota unlimited on DEMO_TBS

```

### 导出数据
从源数据库导出数据，创建的目录要根据实际服务器的目录来。而且要注意oracle用户对于目录是否有读写权限。
```sql
-- 查询
select * from dba_directories;
-- 先创建需要导出的directory（后续导出的dmp会在该目录）
create directory demodump as '/u01/orcl';
```

```bash
# 该操作在服务器oracle用户下执行，无需进入sqlplus
# username 源数据库账号
# password 源数据库密码
# tnsname 如果服务器的客户端没有配置也需要先配置好，否则无法导出。该配置在tnsname.ora文件里
# demodump的路径就是导出的包后续的存放路径
expdp username/password@tnsname schemas=username dumpfile=demodump.dmp directory=demodump  logfile=demo_exp.log
```

### 导入数据
导入到目标数据库，我们这次的目标数据库就是我们新建的oracle数据库。
```sql
-- 查询
select * from dba_directories;
-- 先创建需要导出的directory（后续导入的dmp会在该目录）
create directory demodump as '/u01/orcl';
```
```bash
# 该操作在服务器oracle用户下执行，无需进入sqlplus
# username 源数据库账号
# password 源数据库密码
# tnsname 如果服务器的客户端没有配置也需要先配置好，否则无法导出。该配置在tnsname.ora文件里
# demodump的路径就是导入的包的存放路径
impdp demouser/123456@orcl schemas=demouser directory=demodump dumpfile=demodump.dmp logfile=demo_imp.log
```
注：往往源数据库和目标数据库都不是在同一台服务器上。因此在云数据库导出的dmp包要传输到我们的目标数据库上。如果是高版本数据库导入到低版本，导出的时候最好能加上目标数据的版本号 version=xxxxx。

导入完成后用`demouser/123456`登陆上去看看是否已经成功导入数据。

## 常用命令
```bash
# 切换到oracle用户
su - oracle
# 管理员身份进入命令行
sqlplus / as sysdba
# 启动
startup
# 关闭
shutdown immediate # 推荐
shutdown nomal
shutdown transactional
shutdown abort # 不推荐
```

## 总结
oracle最难装，哪怕是已经装过一遍再来一次，依然每次都会有各式各样的问题。看造化吧。

## 参考链接
1. [Xmanager快速连接Linux图形界面教程](http://www.yelook.com/1858.html)
2. [Linux中使用xmanager安装oracle数据库](https://blog.csdn.net/FIGHT_ANGEL/article/details/47448349)
3. [CentOS 7安装Oracle 12c图文详解](https://www.linuxidc.com/Linux/2017-08/146528.htm)
4. [deepin安装Oracle 12c R2(详细)](https://mackvord.github.io/database/14360.html#%E9%85%8D%E7%BD%AE%E7%8E%AF%E5%A2%83%E5%8F%98%E9%87%8F)
5. [centos安装Oracle之创建swap分区](https://www.jianshu.com/p/5686ea5a4697)
6. [oracle 12c 字符集修改 AL32UTF8 改为 ZHS16GBK](https://blog.csdn.net/antma/article/details/79385497)

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




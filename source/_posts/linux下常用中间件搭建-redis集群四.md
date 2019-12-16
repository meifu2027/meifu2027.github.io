---
title: linux下常用中间件搭建 - redis集群(四)
categories: [原创, 教程]
toc: true
date: 2019-07-05 16:28:49
tags: [linux,redis,集群]
---

## 概况
**redis版本：** 3.2.12  
**redis安装包：** redis-3.2.12.tar.gz  
<!--more-->
**依赖ruby环境：**  
> zlib-1.2.8.tar.gz  
> ruby-1.9.3.tar.gz  
> rubygems-1.8.16.tgz  
> redis-3.0.0.gem  

**主从服务器（2台）：**  
> A：192.168.0.1 （主）节点端口7000、7001、7002  
> B：192.168.0.2 （从）节点端口7003、7004、7005  
> 后续安装ruby环境只需要主服务器需要，会标出。  
**如果只有1台**，redis服务节点还是6个，下面所有需要在两台服务器上分别操作的步骤在这台服务器上都要执行一遍。



## 步骤
### A主机安装zlib
> B主机最好也安装一下，否则后续安装Nginx的时候也需要装。

```bash
tar -xvf /u01/setup/zlib-1.2.8.tar.gz -C /u01
cd /u01/zlib-1.2.8/
./configure
make && make install
```


### A主机安装ruby


```bash
# 先查看是否已经有了
ruby -v
# 有的话且版本低于1.9的可以先卸载掉再装
yum remove ruby
# 解压、编译
tar -zxvf /u01/setup/ruby-1.9.3.tar.gz -C /u01
cd /u01/ruby-1.9.3-p551/
./configure
make && make install
# 再查看是否安装成功，版本应该是：ruby 1.9.3p551 (2014-11-13 revision 48407) [x86_64-linux]
ruby -v
# 如果上条指令报错（-bash: /usr/bin/ruby: 没有那个文件或目录），
# 可能是默认安装路径是/usr/local/bin/ruby 这样可以创建一个软链接解决
ln -s /usr/local/bin/ruby /usr/bin/ruby

```
### A主机安装rubygem

```bash
tar -xvf /u01/setup/rubygems-1.8.16.tgz -C /u01
cd /u01/rubygems-1.8.16
ruby setup.rb
```

### A主机安装gem-redis

```bash
gem install -l /u01/setup/redis-3.0.0.gem
```

### A、B主机安装redis
 

```bash
# 解压、改目录名
tar -zxvf /u01/setup/redis-3.2.12.tar.gz -C /u01
mv /u01/redis-3.2.12 /u01/redis
# 编译
cd /u01/redis
make && make install
# A需要执行、B可执行可不执行
cp /u01/redis/src/redis-trib.rb /usr/local/bin/
# 创建集群目录
mkdir /u01/redis-cluster
cd /u01/redis-cluster
# A新建7000、7001、7002文件夹
mkdir 7000 7001 7002
# B新建7003、7004、7005文件夹
mkdir 7003 7004 7005
```

本来共同的配置应该以单独的配置引入的方式`vim /u01/redis-cluster/redis-common.conf`，但是执行过程中无法读到该文件，因此需要每个文件配置共同的一段配置和基于端口的个性化配置。（后续还是考虑提取公共配置）  
**以7000端口为例：**
```bash
# 新增文件
vim /u01/redis-cluster/7000/redis-7000.conf

# 共同配置部分（直接复制）
daemonize yes
protected-mode no
tcp-backlog 511
timeout 0
tcp-keepalive 300
daemonize yes
supervised no
loglevel notice
logfile ""
databases 16
save 900 1
save 300 10
save 60 10000
stop-writes-on-bgsave-error yes
rdbcompression yes
rdbchecksum yes
dbfilename dump.rdb
dir ./
slave-serve-stale-data yes
slave-read-only yes
repl-diskless-sync no
repl-diskless-sync-delay 5
repl-disable-tcp-nodelay no
slave-priority 100
appendonly no
appendfilename "appendonly.aof"
appendfsync everysec
no-appendfsync-on-rewrite no
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
aof-load-truncated yes
lua-time-limit 5000
cluster-enabled yes
cluster-node-timeout 10000
slowlog-log-slower-than 10000
slowlog-max-len 128
latency-monitor-threshold 0
notify-keyspace-events ""
hash-max-ziplist-entries 512
hash-max-ziplist-value 64
list-max-ziplist-size -2
list-compress-depth 0
set-max-intset-entries 512
zset-max-ziplist-entries 128
zset-max-ziplist-value 64
hll-sparse-max-bytes 3000
activerehashing yes
client-output-buffer-limit normal 0 0 0
client-output-buffer-limit slave 256mb 64mb 60
client-output-buffer-limit pubsub 32mb 8mb 60
hz 10
aof-rewrite-incremental-fsync yes

# 个性化配置(修改下端口和ip)
port 7000
bind 192.168.0.1
pidfile /var/run/redis_7000.pid
cluster-config-file  /u01/redis-cluster/nodes_7000.conf
```

#### A、B主机启动所有节点

```bash
# A主机执行
redis-server /u01/redis-cluster/7000/redis-7000.conf
redis-server /u01/redis-cluster/7001/redis-7001.conf
redis-server /u01/redis-cluster/7002/redis-7002.conf
# B主机执行
redis-server /u01/redis-cluster/7003/redis-7003.conf
redis-server /u01/redis-cluster/7004/redis-7004.conf
redis-server /u01/redis-cluster/7005/redis-7005.conf
# 查看是否启动成功
ps -ef | grep redis
```

#### A主机开启集群配置

```bash
# 若启动成功则会看到图1（输入yes)
ruby /u01/redis/src/redis-trib.rb  create  --replicas  1 192.168.0.1:7000 192.168.0.1:7001 192.168.0.1:7002 192.168.0.2:7003 192.168.0.2:7004 192.168.0.2:7005
# A主机可查看集群当前状态（任意选择一个节点）
redis-trib.rb check 192.168.0.1:7000
```
图1  
![1.png](https://i.loli.net/2019/07/05/5d1f16c938e0e79458.png)








### A、B主机后续配置
```bash
# 修改系统用户级最大open files table的文件限制，解决大并发
ulimit -n 8192
# 开启系统设置TCP连接中TIME-WAIT sockets的快速回收
echo 1 > /proc/sys/net/ipv4/tcp_tw_reuse
```

### 客户端操作

```bash
# ip和端口视实际情况而定
redis-cli -c -p 7000 -h 192.168.0.1
# 查看当前节点下的缓存
192.168.0.1:7000> keys *
# 清空当前节点下所有数据库的缓存
192.168.0.1:7000> flushall

# 注：需分别清除所有主节点的缓存才算清理完成
```


## 总结
redis的搭建可能会在主服务器搭建ruby环境会遇到点问题，一定要仔细，想想前后操作的顺序。
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
*redis 后续还有优化的部分也会更新上来的。*

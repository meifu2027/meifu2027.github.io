---
title: linux下常用中间件搭建 - zookeeper集群(七)
categories: [原创, 教程]
toc: true
date: 2019-07-09 13:01:50
tags: [linux, zookeeper, 集群]
---

## 概况
**zookeeper版本：** 3.4.6  
**zookeeper安装包：** zookeeper-3.4.6.tar.gz  
<!--more-->
**其他依赖环境：**  
> jdk环境（前面章节已安装）


**服务器（2台）**  
> A：192.168.0.1（4个节点：2181 2182 2183 2184）  
> B：192.168.0.2（3个节点：2185 2186 2187）  
> 所有配置两台服务器都要配置。  
**如果只有1台**，建议配置3个集群节点即可。

***单节点的zookeeper很简单，不需要改配置直接启动就行了。生产上一般使用集群配置，至于需要真集群还是伪集群，视情况而定。本文属于伪集群。***

## 步骤
### 解压、基本配置


```bash
# 解压
tar -xzvf /u01/setup/zookeeper-3.4.6.tar.gz  -C /u01


```
### 集群配置

```bash
cd /u01/zookeeper-3.4.6/data
# A主机创建集群节点目录
mkdir -p node2181/log node2182/log node2183/log node2184/log
# B主机创建集群节点目录
mkdir -p node2185/log node2186/log node2187/log
# 每个节点需要增加一个配置，以2181为例
vim /u01/zookeeper-3.4.6/conf/zoo2181.cfg

# 配置内容（注意：所有集群 server.数字 的配置都一样）
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/u01/zookeeper-3.4.6/data/node2181
dataLogDir=/u01/zookeeper-3.4.6/data/node2181/log
clientPort=2181
server.1=192.168.0.1:2881:3881
server.2=192.168.0.1:2882:3882
server.3=192.168.0.1:2883:3883
server.4=192.168.0.1:2884:3884
server.5=192.168.0.2:2885:3885
server.6=192.168.0.2:2886:3886
server.7=192.168.0.2:2887:3887


# A主机每个集群节点在dataDir目录下增加myid配置
echo 1 > /u01/zookeeper-3.4.6/data/node2181/myid
echo 2 > /u01/zookeeper-3.4.6/data/node2182/myid
echo 3 > /u01/zookeeper-3.4.6/data/node2183/myid
echo 4 > /u01/zookeeper-3.4.6/data/node2184/myid

# B主机每个集群节点在dataDir目录下增加myid配置
echo 5 > /u01/zookeeper-3.4.6/data/node2185/myid
echo 6 > /u01/zookeeper-3.4.6/data/node2186/myid
echo 7 > /u01/zookeeper-3.4.6/data/node2187/myid
```

### 启动脚本

由于集群节点比较多，因此需要一个一键启动的脚本（两台服务器都可以放同一个脚本）

```bash
# 新建启动脚本
vim /u01/.zookeeperStart.sh
```


```bash
#!/bin/bash
source /etc/profile
zoo_base=/u01/zookeeper-3.4.6
zkserver=$zoo_base/bin/zkServer.sh
zk_conf=$zoo_base/conf/
servip=`/sbin/ip add |grep -oP '(?<=inet )\S+(?=/)' | grep -v "127.0.0.1"  |sort -n | head -n 1|sed 's/\./-/g'`
if [ "$servip" = "192-168-0-1" ] ; then
    $zkserver stop  $zk_conf/zoo2181.cfg
    $zkserver start $zk_conf/zoo2181.cfg
    $zkserver stop  $zk_conf/zoo2182.cfg
    $zkserver start $zk_conf/zoo2182.cfg
    $zkserver stop  $zk_conf/zoo2183.cfg
    $zkserver start $zk_conf/zoo2183.cfg
    $zkserver stop  $zk_conf/zoo2184.cfg
    $zkserver start $zk_conf/zoo2184.cfg
else
    $zkserver stop  $zk_conf/zoo2185.cfg
    $zkserver start $zk_conf/zoo2185.cfg
    $zkserver stop  $zk_conf/zoo2186.cfg
    $zkserver start $zk_conf/zoo2186.cfg
    $zkserver stop  $zk_conf/zoo2187.cfg
    $zkserver start $zk_conf/zoo2187.cfg
fi

```


```bash
# 启动zookeeper服务
sh /u01/.zookeeperStart.sh 
# 查看是否有对应数量的服务启动了
ps -ef|grep zookeeper

# 另：zookeeper常用操作
/u01/zookeeper-3.4.6/bin/zkServer.sh start
/u01/zookeeper-3.4.6/bin/zkServer.sh stop
/u01/zookeeper-3.4.6/bin/zkServer.sh status

```





## 总结
有优化配置后续再补充。对于zookeeper集群的更详细介绍网上也有很多，有需要可以自己再找。
[《构建高可用zookeeper集群》](https://www.cnblogs.com/cyfonly/p/5626532.html)





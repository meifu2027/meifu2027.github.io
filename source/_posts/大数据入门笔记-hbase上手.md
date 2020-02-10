---
title: 大数据入门笔记-hbase上手
categories: [原创, 笔记]
toc: true
date: 2020-02-06 23:04:23
tags: [大数据, hbase]
---
学习下分布式结构化数据库HBase。
<!--more-->
## HBASE版本
[hbase-2.2.3-bin.tar.gz](http://mirror.bit.edu.cn/apache/hbase/2.2.3/hbase-2.2.3-bin.tar.gz)(该版本对应jdk版本为1.8，查看版本对应关系看[这里](https://hbase.apache.org/book.html#java))
```bash
# 按照国际惯例还是下到 /u01/setup目录下
cd /u01/setup
wget http://mirror.bit.edu.cn/apache/hbase/2.2.3/hbase-2.2.3-bin.tar.gz
tar -zxvf /u01/setup/hbase-2.2.3-bin.tar.gz -C /u01
mv /u01/hbase-2.2.3 /u01/hbase
```
## 新建用户、授权
```bash
groupadd hbase
useradd -g hbase hbase
echo "your_password" | passwd --stdin hbase
chown -R hbase:hbase /u01/hbase
# 切换至hbase用户
su - hbase
```
## 单节点启动
1. 在hbase的配置文件中配置JAVA_HOME
```bash
vim /u01/hbase/conf/hbase-env.sh
# 增加配置 /u01/jdk1.8.0_201 是我的服务器上的配置
export JAVA_HOME=/u01/jdk1.8.0_201
```
2. 编辑HBase的主要配置文件 `/u01/hbase/conf/hbase-site.xml`。指定一个系统路径以便HBase、Zookeeper可以写入数据和确认风险。虽然默认的路径为`/tmp`，但是由于很多服务的默认路径都是`/tmp`，且重启后会删除该目录下文件，因此需要更换路径。以下配置会在让HBase的数据存放在 `/home/hbase/hbase`目录下。
```bash
vim /u01/hbase/conf/hbase-site.xml
# 改成
<configuration>
  <property>
    <name>hbase.rootdir</name>
    <value>file:///home/hbase/hbase</value>
  </property>
  <property>
    <name>hbase.zookeeper.property.dataDir</name>
    <value>/home/hbase/zookeeper</value>
  </property>
  <property>
    <name>hbase.unsafe.stream.capability.enforce</name>
    <value>false</value>
    <description>
      Controls whether HBase will check for stream capabilities (hflush/hsync).

      Disable this if you intend to run on LocalFileSystem, denoted by a rootdir
      with the 'file://' scheme, but be mindful of the NOTE below.

      WARNING: Setting this to false blinds you to potential data loss and
      inconsistent system state in the event of process and/or node failures. If
      HBase is complaining of an inability to use hsync or hflush it is most
      likely not a false positive.
    </description>
  </property>
</configuration>
```
**注意：**你无需事先建好HBase的数据目录，HBase会自己建好的，如果你创建了该目录，HBase会做一个迁移，这与你的初衷背道而驰。
> 上述配置中的`hbase.rootdir`指向了本地文件系统的目录，前缀`file://`就是为了表示是本地文件系统。你应该牢记上述配置中WARNING的内容。在单节点模式中HBase使用Apache Hadoop项目中的本地文件系统抽象。这种抽象不能提供HBase安全运行所需的持久性保证。这对于本地开发很运行测试用例很好，避免了分布式失败带来的时间成本。这种方式不适合生产环境，最终会丢失数据。

要将HBase放在现有HDFS实例上，请将hbase.rootdir设置为指向实例上的目录，比如：`hdfs://meifu:9000/user/hbase` 其中hdfs中hbase目录需要自己重新建一下，hadoop上手的笔记中有说过，可以看下。有关此变体的更多信息，请参见下面有关基于HDFS的独立HBase的部分[Standalone HBase over HDFS](https://hbase.apache.org/book.html#standalone_dist)

1. `/u01/hbase/bin/start-hbase.sh`提供了便捷的启动方式，执行后会有打印日志告诉你启动成功，可以使用`jps`命令检验启动的程序名称为`HMaster`,然后打开`http://localhost:16010 `你就可以看到HBase的web页面（云服务器的端口记得开放）。
![20200207-1.png](/img/blog/20200207-1.png)
![20200207-2.png](/img/blog/20200207-2.png)
![20200207-3.png](/img/blog/20200207-3.png)

2. 打开hbase命令行
```bash
cd /u01/hbase
./bin/hbase shell
```
3. 打开后若要显示HBase的帮助文档可直接输入`help`
4. 建表 `create 'test', 'cf'`
5. 查看表`list 'test'`;使用`describe 'test'`可查看更多细节;
6. 将数据插入表中
```bash
# test为表，row1表示第1行，cf表示列族，a表示列限定符，value1为值；默认还会插入时间戳，也可以自己插入
put 'test', 'row1', 'cf:a', 'value1'
put 'test', 'row2', 'cf:b', 'value2'
put 'test', 'row3', 'cf:c', 'value3'
```
7. 可使用`scan 'test'`一次性查看表中所有的数据，也可以限定条件，具体可以看用法`help "scan"`
8. 获取单行数据`get 'test', 'row1'`
9. 使一张表失效或生效 `disable 'test'` `enable 'test'`
10. 删除表`drop 'test'`（需要先使表失效）
11. 退出HBase命令行`quit`
12. 停止HBase运行 `/u01/hbase/bin/stop-hbase.sh` ;停止后可再次使用`jps`命令确认`HMaster`和`HRegionServer`确实已关闭

## 伪分布式启动
1. 若上述单节点HBase还在运行，则需要停止，且我认为你已经启动了hadoop了。最后有我hadoop的笔记链接。
2. 修改HBase的分布式配置`vim /u01/hbase/conf/hbase-site.xml`
```xml
<property>
  <name>hbase.cluster.distributed</name>
  <value>true</value>
</property>
<!--目录同样无需提前设置，hbase自己会建目录-->
<!--这里用localhost而不用meifu是因为meifu会被自己之前配置的域名解析为0.0.0.0
    而localhost 是127.0.0.1，可以找到hdfs的服务ip，也就是本机-->
<property>
  <name>hbase.rootdir</name>
  <value>hdfs://localhost:9000/hbase</value>
</property>
```
**注意：**确保没有`hbase.unsafe.stream.capability.enforce`属性配置或将其属性值设置为`true`；另外，此处需要解决hbase用户在hadoop用户下写权限问题，否则会出现hadoop客户端往hadoop写文件的同样的权限的报错导致目录无法正常生成。
3. HBase应用也存在需要免密登录的需求，因此可以执行
```bash
ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
chmod 0600 ~/.ssh/authorized_keys
```
3. 启动HBase，`/u01/hbase/bin/start-hbase.sh` 同样用`jps`查看启动情况
![20200207-4.png](/img/blog/20200207-4.png)
4. 命令查看HBase运行文件是否已生成`./bin/hadoop fs -ls /hbase`
![20200207-5.png](/img/blog/20200207-5.png)
5. 其实启动、hbase shell、数据库增删改查、关闭服务都同上述单节点1~12步骤一样的；
6. 启动、关闭HBase Master的备份服务。最多可以再另外启动9个HMaster备份服务，默认使用的端口为`16000`和`16010`，启动的时候可以通过设置偏移量的方式指定端口和数量，例：
```bash
cd /u01/hbase
# 启动的3个服务端口为`16002``16012`、`16003``16013`、`16005``16015`
./bin/local-master-backup.sh start 2 3 5
# 启动后关闭
# 固定的格式为 /tmp/hbase-USER-X-master.pid
cat /tmp/hbase-hbase-2-master.pid |xargs kill -9
cat /tmp/hbase-hbase-3-master.pid |xargs kill -9
cat /tmp/hbase-hbase-5-master.pid |xargs kill -9
```
7. 启停额外的RegionServers与上述类似，默认端口`16020`和`16030`。若10个偏移量不够，还可以自己在文件`local-regionservers.sh`中设置`HBASE_RS_BASE_PORT`和`HBASE_RS_INFO_BASE_PORT`变量指定最多99个额外的RegionServers,此处的默认端口为`16200`和`16300`
```bash
cd /u01/hbase
# 启动
./bin/local-regionservers.sh start 2 3 4 5
# 停止
./bin/local-regionservers.sh stop 2 3 4 5
```
8. 停止HBase命令同单节点`/u01/hbase/bin/stop-hbase.sh`

## 分布式运行
因为只有一台服务器没办法搞真实的分布式，所以有条件的同学请参照分布式配置[Advanced - Fully Distributed](https://hbase.apache.org/book.html#quickstart_fully_distributed)

## 相关链接
[**大数据技术原理与应用**](https://study.163.com/course/courseMain.htm?courseId=1002887002)
[**Quick Start - Standalone HBase**](https://hbase.apache.org/book.html#quickstart)
[**大数据入门笔记-hadoop上手**](/2020/02/05/大数据入门笔记-hadoop上手/)
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
mv /u01/hbase-2.2.3-bin /u01/hbase
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
12. 停止HBase运行 `./bin/stop-hbase.sh` ;停止后可再次使用`jps`命令确认`HMaster`和`HRegionServer`确实已关闭

## 伪分布式启动


## 相关链接
[**大数据技术原理与应用**](https://study.163.com/course/courseMain.htm?courseId=1002887002)
[**Quick Start - Standalone HBase**](https://hbase.apache.org/book.html#quickstart)
[**大数据入门笔记-hadoop上手**](/2020/02/05/大数据入门笔记-hadoop上手/)
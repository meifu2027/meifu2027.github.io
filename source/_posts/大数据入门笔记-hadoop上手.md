---
title: 大数据入门笔记-hadoop上手
categories: [原创, 笔记]
toc: true
date: 2020-02-05 11:38:18
tags: [大数据, hadoop]
---
受疫情影响，我被困在家里不能出门了。学习大数据一方面是想以后从java web转型到另一个领域，另外一方面我也在思考：国家危难之际，我能为社会做点什么贡献？相比而言，大数据可能对于社会的作用更加的明显，那就开始愉快地学习吧。*以下内容主要参考自[Hadoop官网](http://hadoop.apache.org/)教程。*
<!--more-->
## 目的
本文档主要是加深印象并记录搭建过程中一些遇到的问题，供自己查看。
## 先决条件
### 平台
本案例使用**腾讯云服务器**，操作系统为`CentOS Linux release 7.6.1810 (Core) `
### 必备软件
* Java (本案例使用版本`jdk-8u201-linux-x64.tar.gz`)
* ssh (这个不用说了，服务器已自带了)

## 下载
[**官方仓库**](http://mirror.bit.edu.cn/apache/hadoop/common/)
我选择的版本(按照以往的尿性我还是放在了`/u01/setup`目录下)  
[**hadoop-3.2.1.tar.gz**](http://mirror.bit.edu.cn/apache/hadoop/common/hadoop-3.2.1/hadoop-3.2.1.tar.gz)
下载完后可以解压到 `/u01`目录下
```bash
tar -zxvf /u01/setup/hadoop-3.2.1.tar.gz -C /u01
mv /u01/setup/hadoop-3.2.1 /u01/setup/hadoop
```
## 创建hadoop用户（官网教程没有这一步）
```bash
# 创建用户组
groupadd hadoop
# 创建用户
useradd -g hadoop hadoop
# 设置密码
echo "your_password" | passwd --stdin hadoop
# 目录授权
chown -R hadoop:hadoop /u01/hadoop
# 切换用户
su - hadoop
```
**注意：**后续操作都在hadoop用户下执行。
## 准备启动Hadoop集群
修改配置 
```bash
vim /u01/hadoop/etc/hadoop/hadoop-env.sh
# 增加JAVA_HOME路径
export JAVA_HOME=/u01/jdk1.8.0_201
# 保存退出后 尝试执行以下命令，若正常则可以看到类似如下图片信息
/u01/hadoop/bin/hadoop
```
![20200205-1.png](/img/blog/20200205-1.png)
**注意：**若系统已经配置JAVA_HOME路径（可以使用`echo $JAVA_HOME`查看），也可以不再修改上述文件，但是一定要确保**hadoop版本和java版本对应**起来，具体版本对应最后有链接补充说明。本案例中hadoop3.2.1对应jdk1.8版本。  
## 单机运行

```bash
cd /u01/hadoop
mkdir input
cp etc/hadoop/*.xml input
bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-3.2.1.jar grep input output 'dfs[a-z.]+'
cat output/*
# 能有正常输出就说明没有问题
```
## 伪分布式运行
### 配置
`vim /u01/hadoop/etc/hadoop/core-site.xml`
```xml
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
</configuration>
```
`vim /u01/hadoop/etc/hadoop/hdfs-site.xml`
```xml
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
</configuration>
```
### 设置ssh免密登录
```bash
# 如果提示你需要输入密码则配置下免密
ssh localhost
# 配置免密
ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
chmod 0600 ~/.ssh/authorized_keys
# 配置好再试下应该不需要再输入密码了
```
### 执行
1. 格式化文件系统（只需要执行一次即可） 
```bash
/u01/hadoop/bin/hdfs namenode -format
```
2. 启动NameNode和DataNode的守护进程（可以自定义环境变量`$HADOOP_LOG_DIR`指定日志输出路径，默认路径为`$HADOOP_HOME/logs`）
```bash
/u01/hadoop/sbin/start-dfs.sh
```
3. 浏览器查看NameNode - `http://localhost:9870/` (注意：腾讯云服务器需要在控制台开通访问端口)
![20200205-2.png](/img/blog/20200205-2.png)
4. 创建HDFS目录，以执行MapReduce任务
```bash
cd /u01/hadoop
bin/hdfs dfs -mkdir /user
# bin/hdfs dfs -mkdir /user/<username>
bin/hdfs dfs -mkdir /user/hadoop
```
5. 拷贝文件到HDFS的input目录中去
```bash
cd /u01/hadoop
bin/hdfs dfs -mkdir input
bin/hdfs dfs -put etc/hadoop/*.xml input
```
6. 运行hadoop提供的示例
```bash
cd /u01/hadoop
bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-3.2.1.jar grep input output 'dfs[a-z.]+'
```
7. 检查输出文件
```bash
cd /u01/hadoop
# 将HDFS中的文件拷贝到本地并检查
bin/hdfs dfs -get output output
cat output/*
# 或直接在HDFS中查看
bin/hdfs dfs -cat output/*
```
8. 如果你不想玩了，关掉的命令是
```bash
/u01/hadoop/sbin/stop-dfs.sh
```
### 单节点运行YARN
上述步骤的1~4需要确保已经执行过了
1. 修改配置
```bash
cd /u01/hadoop
vim etc/hadoop/mapred-site.xml
# 改成
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
    <property>
        <name>mapreduce.application.classpath</name>
        <value>$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/*:$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/lib/*</value>
    </property>
</configuration>
# 保存退出后再修改另一个文件
vim etc/hadoop/yarn-site.xml
# 改成
<configuration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <property>
        <name>yarn.nodemanager.env-whitelist</name>
        <value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAPRED_HOME</value>
    </property>
</configuration>
```
2. 启动 
```bash
/u01/hadoop/sbin/start-yarn.sh
```
3. 浏览器查看资源管理界面 - `http://localhost:8088/` (注意：腾讯云服务器需要在控制台开通访问端口)
![20200205-3.png](/img/blog/20200205-3.png)
4. 启动一个MapReduce任务
5. 玩够了，想关了 
```bash
/u01/hadoop/sbin/stop-yarn.sh
```
## 分布式运行
因为只有一台服务器没办法搞真实的分布式，所以有条件的同学请参照分布式配置[Hadoop Cluster Setup](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/ClusterSetup.html)
## 参考链接
[*Hadoop: Setting up a Single Node Cluster*](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/SingleCluster.html)
[*Hadoop Java Versions*](https://cwiki.apache.org/confluence/display/HADOOP/Hadoop+Java+Versions)
[*大数据技术原理与应用*](https://study.163.com/course/courseMain.htm?courseId=1002887002)

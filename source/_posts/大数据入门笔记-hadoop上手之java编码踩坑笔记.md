---
title: 大数据入门笔记-hadoop上手之java编码踩坑笔记
categories: [原创, 笔记]
toc: true
date: 2020-02-06 16:38:40
tags: [大数据, hadoop, java]
---
开始上手简单编程过程中踩到一些坑，花了好些时间去解决，基本上问题都是由于我使用的是腾讯的云服务器引起的，若开发和hadoop部署是同一台机器能避免其中的好多问题，我想这大概也是我看的课程中老师要求学生把伪集群搭建和eclipse开发放在同一台服务器上的主要原因吧。仅此记录。
<!--more-->
## Hadoop 9000端口无法访问
### 问题描述
**在腾讯云服务器已经放行9000端口的前提下，本地开发无法访问到9000端口**
### 解决办法
**原因：**
`netstat -nptl` 查看端口
![**20200206-1.png**](/img/blog/20200206-1.png)
看到9000端口已经启动，但是只能`127.0.0.1`也就是本机才能访问。还记得我们最初配置的`vim /u01/hadoop/etc/hadoop/core-site.xml` 这个文件吗？这里有`hdfs://localhost:9000`,就是这个`localhost`搞的鬼。  
**方案：**
* hadoop用户 先把 `hdfs://localhost:9000` 改成 `hdfs://meifu:9000` （这个meifu可以是任何你自己想改的）；
* root用户 增加域名解析 `vim /etc/hosts`，增加`0.0.0.0 meifu`；
* root用户 重启网卡 `service network restart`；
* hadoop用户 关闭hdfs `/u01/hadoop/sbin/stop-dfs.sh` 启动hdfs `/u01/hadoop/sbin/start-dfs.sh`；
* 再次查看端口 `netstat -nptl` 9000 对应的访问权限就是0.0.0.0了；
* 客户端再次访问9000端口，没有问题；

## bin/hdfs dfs -put etc/hadoop/*.xml input 出错
### 问题描述
重新格式化文件系统后再次执行之前步骤，复制文件到HDFS中去的时候出错；
### 解决办法
**原因：**临时目录文件有冲突；
**方案：**
```bash
# 切换到hadoop用户
su - hadoop
# 删除能删的临时目录
rm -rf /tmp/*
# 再次put应该没问题了
```

## HDFS web端下载文件出错
### 问题描述
![**20200206-2.png**](/img/blog/20200206-2.png)
![**20200206-3.png**](/img/blog/20200206-3.png)
![**20200206-4.png**](/img/blog/20200206-4.png)
### 解决办法
**原因：**`_`属于http请求中的非法字符，也就是说我服务器的主机名有问题；
**方案：**  
* 那就改主机名吧`hostnamectl set-hostname meifu2027`
* 既然请求中有主机名，那么本地必须要做域名解析，否则无法解析域名，修改hosts文件（我的windows下目录是`C:\Windows\System32\drivers\etc\hosts`，需管理员权限），最后增加 `192.168.0.1 meifu2027` (请输入你的远程服务器的ip)；
* 刷新DNS 打开cmd  输入`ipconfig /flushdns`；
* 重新下载成功；
**注意：**云服务器安全组中9864端口也需要允许访问。
## 本地开发java客户端下载文件出错
### 问题描述
读文件，能判断到文件存在，但是读取的时候报错 `org.apache.hadoop.hdfs.BlockMissingException: Could not obtain block`
### 解决办法
**原因：**hdfs的block记录的是服务器的内网地址，而我们只能访问到服务器的外网地址，因此无法访问到目标block；
**方案：**增加客户端读取主机名配置（要搭配上一问题中的本地域名解析一起使用，否则仅仅是主机名还是无法到达目标服务器）
增加的**客户端**文件`hdfs-site.xml`配置；
```xml
<property>
    <name>dfs.client.use.datanode.hostname</name>
    <value>true</value>
    <description>only cofig in clients</description>
</property>
```
代码方式：`conf.set("dfs.client.use.datanode.hostname","true");`

## 本地开发java往远程HDFS写文件出错
### 问题描述
往远程写文件的时候报错。
`org.apache.hadoop.security.AccessControlException: Permission denied: user=pc_user, access=WRITE, inode="/user":hadoop:supergroup:drwxr-xr-x`
### 解决办法
**原因：**写文件需要hadoop用户的写权限，但是客户端显然不是同一台机器，自然不会有权限。
**方案：**客户端开发暂无。若是部署在服务器上的情况，应该没有问题的。

## 本地开发引入jar包说明
本地开发同样需要下载hadoop文件，然后引入其中的jar包，或者通过maven依赖的方式？
**以下是我所有的依赖**
![20200206-5.png](/img/blog/20200206-5.png)
## 相关代码
本代码参考自《大数据技术原理与应用》课程
HDFSFileIfExist.java
```java
package com.meifu.test;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;

public class HDFSFileIfExist {

    public static void main(String[] args){
        try{
            String fileName = "/user/hadoop/input/core-site.xml";
            Configuration conf = new Configuration();
            // // 192.168.0.1为远程主机ip
            conf.set("fs.defaultFS", "hdfs://192.168.0.1:9000");
            conf.set("fs.hdfs.impl", "org.apache.hadoop.hdfs.DistributedFileSystem");
            FileSystem fs = FileSystem.get(conf);
            if(fs.exists(new Path(fileName))){
                System.out.println("文件存在");
            }else{
                System.out.println("文件不存在");
            }
        }catch (Exception e){
            e.printStackTrace();
        }
    }
}
```
WriteFile.java
```java
package com.meifu.test;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FSDataOutputStream;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;

public class WriteFile {

    public static void main(String[] args) {
        try {
            Configuration conf = new Configuration();
            // 192.168.0.1为远程主机ip
            conf.set("fs.defaultFS","hdfs://192.168.0.1:9000");
            conf.set("fs.hdfs.impl","org.apache.hadoop.hdfs.DistributedFileSystem");
            FileSystem fs = FileSystem.get(conf);
            byte[] buff = "Hello world".getBytes(); // 要写入的内容
            String filename = "test.txt"; //要写入的文件名
            FSDataOutputStream os = fs.create(new Path(filename));
            os.write(buff,0,buff.length);
            System.out.println("Create:"+ filename);
            os.close();
            fs.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```
ReadFile.java
```java
package com.meifu.test;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FSDataInputStream;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import java.io.BufferedReader;
import java.io.InputStreamReader;

public class ReadFile {

    public static void main(String[] args) {
        try {
            Configuration conf = new Configuration();
            // 192.168.0.1为远程主机ip
            conf.set("fs.defaultFS","hdfs://192.168.0.1:9000");
            conf.set("fs.hdfs.impl","org.apache.hadoop.hdfs.DistributedFileSystem");
            // 增加使用主机名配置（课程代码中没有该行）
            conf.set("dfs.client.use.datanode.hostname","true");
            FileSystem fs = FileSystem.get(conf);
            Path file = new Path("/user/hadoop/input/hadoop-policy.xml");
            FSDataInputStream getIt = fs.open(file);
            BufferedReader d = new BufferedReader(new InputStreamReader(getIt));
            String content = d.readLine(); //读取文件一行
            System.out.println(content);
            d.close(); //关闭文件
            fs.close(); //关闭hdfs
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```
## 相关链接
[**大数据入门笔记-hadoop上手**](/2020/02/05/大数据入门笔记-hadoop上手/)
[**大数据技术原理与应用 第三章 分布式文件系统HDFS 学习指南**](http://dblab.xmu.edu.cn/blog/290-2/)
[**Hadoop本地开发，9000端口拒绝访问**](https://blog.csdn.net/yjc_1111/article/details/53817750)
[**Hadoop Illegal character in host-name**](https://github.com/rancher/community-catalog/issues/248)
[**阿里云搭建hadoop集群服务器，内网、外网访问问题（详解。。。）**](https://www.cnblogs.com/ya-qiang/p/10076424.html)
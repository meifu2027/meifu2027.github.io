---
title: linux下常用中间件搭建 - jdk安装(三)
categories: [原创, 教程]
toc: true
date: 2019-07-05 09:33:27
tags: [linux,jdk]
---

## 概况
**jdk版本：** 1.7.0_67  
**jdk安装包：** jdk1.7.0_67.tar.gz
<!--more-->
## 步骤
### 解压、更名
```bash
# 解压到/u01目录下
tar -zxvf /u01/setup/jdk1.7.0_67.tar.gz  -C /u01
# 解压完后重命名为jdk
mv /u01/jdk1.7.0_67 /u01/jdk
```

### 配置环境变量

```bash
# 一般linux都会自带openjdk，我们可以看下版本（图1）
java -version
# 修改环境变量
vim /etc/profile
# 在profile文件中添加配置（图2）
export JAVA_HOME=/u01/jdk
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
# 使配置生效
source /etc/profile
# 再次查看java版本（图3）
java -version
```

图1  
![1原版本.png](https://i.loli.net/2019/07/05/5d1eabb140efc77166.png)
图2  
![配置环境变量.png](https://i.loli.net/2019/07/05/5d1eabb17e4c963076.png)  
图3  
![3安装后版本.png](https://i.loli.net/2019/07/05/5d1eabb1696fa81506.png)


## 总结
*jdk的离线安装相对简单*
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

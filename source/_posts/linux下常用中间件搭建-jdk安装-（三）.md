---
title: linux下常用中间件搭建 - jdk安装 （三）
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
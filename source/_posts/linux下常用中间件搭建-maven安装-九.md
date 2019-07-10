---
title: linux下常用中间件搭建 - maven安装(九)
categories: [原创, 教程]
toc: true
date: 2019-07-10 12:20:14
tags: [linux, maven]
---

## 概况
**maven版本：** 5.12.0  
**maven安装包：** apache-maven-3.5.0.zip  
<!--more-->
**其他依赖环境：**  
> jdk环境（前面章节已安装）


**服务器（1台）**  
> A：192.168.0.1  
> B：192.168.0.2   
> 配合后续git完成打包发布，任意选择一台安装即可。



## 步骤
### 解压


```bash
# 解压
unzip apache-maven-3.5.0.zip -d /u01
```
### 配置环境变量
```bash
# 这里配置到用户的环境变量（系统的环境变量是 /etc/profile）
vim ~/.bash_profile
# 最后增加
export M2_HOME=/u01/apache-maven-3.5.0
export PATH=$M2_HOME/bin:$PATH:/sbin

# 使配置生效
source ~/.bash_profile
# 查看版本（能正常看到就是配置生效了,如图1）
mvn -v
```
图1  
![2.png](https://i.loli.net/2019/07/10/5d2569ff1b2cb38427.png)

由于是在内网环境，因此直接将maven仓库上传至`/u01/.m2report`目录

```bash
# 修改仓库地址
vim /u01/apache-maven-3.5.0/conf/settings.xml
# 修改内容
<localRepository>/u01/.m2report</localRepository>

```




## 总结
mvn的配置比较简单。比较麻烦的是后续如果有新的依赖，内网无法下载镜像仓库的jar包，就只能本地上传。




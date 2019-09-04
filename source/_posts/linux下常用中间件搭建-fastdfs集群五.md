---
title: linux下常用中间件搭建 - fastdfs集群(五)
categories: [原创, 教程]
toc: true
date: 2019-07-08 15:22:44
tags: [linux,fastdfs,集群]
---

## 概况
**fastdfs版本：** 5.11  
**fastdfs安装包：** fastdfs-5.11.tar.gz  
<!--more-->
**其他依赖环境：**  
> libfastcommon-1.0.39.zip  
> fastdfs-nginx-module-1.20.tar.gz  


**主从服务器（2台）：**  
> A：192.168.0.1 （主）一个tracker节点、一个storage节点  
> B：192.168.0.2 （从）1个storage节点
> 后续只有tracker配置的环境，会标出。  
**如果只有1台**，只需要一个storage节点即可。



## 步骤
### A、B主机安装libfastcommon依赖


```bash
mkdir /u01/fastdfs
unzip -n /u01/setup/libfastcommon-1.0.39.zip -d /u01/fastdfs
cd /u01/fastdfs/libfastcommon-1.0.39
./make.sh;./make.sh install
```


### A、B主机安装fastdfs


```bash
tar -xzvf /u01/setup/fastdfs-5.11.tar.gz -C /u01/fastdfs
cd /u01/fastdfs/fastdfs-5.11/
 ./make.sh; ./make.sh install

```
### A、B主机新建用户、授权

```bash
useradd fastdfs -M -s /sbin/nologin
mkdir /u01/fastdfs/data
chown -R fastdfs:fastdfs  /u01/fastdfs/data
```

### A、B主机集群配置

```bash
cp -R /u01/fastdfs/fastdfs-5.11/conf/*  /etc/fdfs/
```

### A主机进行tracker服务器配置
 

```bash
# 修改tracker配置
vim /etc/fdfs/tracker.conf

# 修改内容
base_path=/u01/fastdfs/data
run_by_group=fastdfs
run_by_user=fastdfs
store_lookup = 0

# 修改完后启动
/usr/bin/fdfs_trackerd /etc/fdfs/tracker.conf start

```

### A、B主机进行storage服务器配置
 

```bash
# 修改storage配置
vim /etc/fdfs/storage.conf

# 修改内容
group_name=group1 # 当前只有一个group
run_by_group=fastdfs
run_by_user=fastdfs
base_path=/u01/fastdfs/data
tracker_server=192.168.0.1:22122
store_path0=/u01/fastdfs/data

# 修改完后启动
/usr/bin/fdfs_storaged /etc/fdfs/storage.conf start

# 创建软链接
ln -s /u01/fastdfs/data/data  /u01/fastdfs/data/data/M00

```
### A、B主机配置Monitor（客户端）
```bash
vim /etc/fdfs/client.conf
# 修改内容
base_path=/u01/fastdfs/data
tracker_server=192.168.0.1:22122

```

### A、B主机配置fastdfs-nginx模块文件
```bash
tar -xzvf /u01/setup/fastdfs-nginx-module-1.20.tar.gz -C /u01/fastdfs
cp /u01/fastdfs/fastdfs-nginx-module-1.20/src/mod_fastdfs.conf /etc/fdfs

# 修改配置
vim /etc/fdfs/mod_fastdfs.conf
# 修改内容
tracker_server=192.168.0.1:22122
storage_server_port=23000
group_name=group1
store_path_count=1
store_path0=/u01/fastdfs/data
url_have_group_name = true

# 修改配置
vim /u01/fastdfs/fastdfs-nginx-module-1.20/src/config
# 修改内容
ngx_module_incs="/usr/include/fastdfs /usr/include/fastcommon/"
CORE_INCS="$CORE_INCS /usr/include/fastdfs /usr/include/fastcommon/"

```


### 上传测试


```bash
# 上传/opt目录下的一张测试图片
/usr/bin/fdfs_test  /etc/fdfs/client.conf upload /opt/test.jpg
# 上传成功后会生成一个类似这种格式的url
# http://192.168.0.1/group1/M00/00/00/C-AARlkk8p-ANxWYAAEIe1Ty1ZY285_big.png      
# 现在还不能直接下载，等下一篇配置了nginx后就能下载啦。

```




## 总结
上述倒数第二个步骤是由于fastdfs文件的下载需要依赖nginx，因此该步骤配置完成后需要配合nginx的配置才能生效。在下一章节会说明。  
附上fastdfs架构图  
![1.png](https://i.loli.net/2019/07/08/5d22f6e3ec95f96315.png)
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






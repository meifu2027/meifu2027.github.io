---
title: linux下常用中间件搭建 - nginx搭建(六)
categories: [原创, 教程]
toc: true
date: 2019-07-08 15:58:32
tags: [linux,nginx]
---

## 概况
**nginx版本：** 1.16.0  
**nginx安装包：** nginx-1.16.0.tar.gz  
<!--more-->
**其他依赖环境：**  
> gcc（本人安装环境已具备，此步骤忽略）  
> pcre-devel-7.8-6.el6.x86_64.rpm  
> pcre-static-7.8-6.el6.x86_64.rpm(这个应该是可以不要的，但是我一起装了)    
> zlib（在装redis的时候已经装过了，此步骤忽略）  
> openssl（本人安装环境已具备，此步骤忽略）


**服务器（2台，此nginx配置无主从关系）：**  
> A：192.168.0.1 
> B：192.168.0.2
> 所有配置两台服务器都要配置。  
**如果只有1台**，只需要配置1次即可。

***本文的配置主要针对上一篇的fastdfs下载进行配置。***

## 步骤
### 安装必须依赖


```bash
cd /u01/setup
rpm -ivh pcre*.rpm
```


### 安装nginx


```bash
tar -xzvf /u01/setup/nginx-1.16.0.tar.gz -C /u01
cd /u01/nginx-1.16.0
# 如果不是特定用于fastdfs的下载，"./configure" 即可
./configure --add-module=/u01/fastdfs/fastdfs-nginx-module-1.20/src
make && make install

```
### 修改nginx配置

```bash
# 修改配置
vim /usr/local/nginx/conf/nginx.conf
# 修改内容
server{
	    listen      80;
	    server_name localhost;
        # 在80端口的配置下增加这部分
        location /group1/M00 {
            root   /u01/fastdfs/data;
            ngx_fastdfs_module;
        }
}
```

### nginx常用命令

```bash
# 启动 启动后可以访问 http://192.168.0.1 或http://192.168.0.2
# 查看是否能正常显示nginx相关页面
/usr/local/nginx/sbin/nginx
# Nginx配置后重启
/usr/local/nginx/sbin/nginx -s reload
# 停止 
/usr/local/nginx/sbin/nginx -s stop
# 检查 Nginx 进程
ps aux | grep nginx
# 查看错误日志
tail -200f /usr/local/nginx/logs/error.log
```




## 总结
nginx正常启动后，就能下载上一篇fastdfs的附件啦。后续还会有nginx做端口转发的相关配置。





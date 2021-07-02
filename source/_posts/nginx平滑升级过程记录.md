---
title: nginx平滑升级过程记录
categories:
  - 原创
toc: true
date: 2021-07-02 10:58:18
tags: [nginx]
---
> 对于诸如nginx一类的中间件时常会因为被发现漏洞而升级，今天我来记录下nginx平滑升级的过程。
<!-- more -->

## 一、nginx信号简介
### 主进程支持的信号
- `TERM`, `INT`: 立刻退出
- `QUIT`: 等待工作进程结束后再退出
- `KILL`: 强制终止进程
- `HUP`: 重新加载配置文件，使用新的配置启动工作进程，并逐步关闭旧进程。
- `USR1`: 重新打开日志文件
- `USR2`: 启动新的主进程，实现热升级
- `WINCH`: 逐步关闭工作进程
### 工作进程支持的信号
- `TERM`, `INT`: 立刻退出
- `QUIT`: 等待请求处理结束后再退出
- `USR1`: 重新打开日志文件
## 二、版本查看
使用命令查看nginx版本  
`/usr/local/nginx/sbin/nginx -V` 
![nginx版本查看](/img/blog/20210702-1-nginx版本查看.jpg)
可以看到我的nginx版本是`1.16.1`，且编译参数为`--with-http_stub_status_module --with-http_ssl_module --with-stream` (**非常重要**，每个人的可能不一样，记录下需要升级的nginx的编译参数即可)。前去[*官网下载列表*](https://nginx.org/en/download.html)下载最新的stable版本，截止到今天最新版本为`1.20.1`
![nginx官网列表](/img/blog/20210702-2-nginx官网下载列表.jpg)

## 三、备份配置文件
对于我们项目来说，nginx配置反向代理的配置项是最重要的工作，因此可以先手动备份一下nginx的配置文件到本地  
`sz /usr/local/nginx/conf/nginx.conf`
## 四、重新编译、启动
### 先进行编译
```bash
# 解压最新文件
[root@ESB-Main setup]# tar -zxvf nginx-1.20.1.tar.gz -C /u01
# 到目录当中去
[root@ESB-Main setup]# cd /u01/nginx-1.20.1
# 编译 ./configure后面带上你之前查出来的参数
[root@ESB-Main nginx-1.20.1]# ./configure --with-http_stub_status_module --with-http_ssl_module --with-stream
# 只需要到 make，千万不要 make install 否则会将原配置文件覆盖
[root@ESB-Main nginx-1.20.1]# make
```
### 备份现有nginx二进制文件（nginx不会停止服务）
```bash
[root@ESB-Main nginx-1.20.1]# mv /usr/local/nginx/sbin/nginx /usr/local/nginx/sbin/nginx_$(date +%F)
```
### 复制新的nginx二进制文件到现目录
```bash
[root@ESB-Main nginx-1.20.1]#  cp /u01/nginx-1.20.1/objs/nginx /usr/local/nginx/sbin/
```
### 测试新版本nginx是否正常
```bash
# 输出test is successful 表示没有问题
/usr/local/nginx/sbin/nginx -t
```
![新版本nginx二进制文件测试](/img/blog/20210702-3-新版本nginx二进制文件测试.jpg)
### 给nginx发送平滑迁移信号
```bash
# 查看nginx配置文件中的pid路径，我这里默认是 /usr/local/nginx/logs/nginx.pid
[root@ESB-Main nginx-1.20.1]# cat /usr/local/nginx/conf/nginx.conf |grep pid
#pid        logs/nginx.pid;
# 发送迁移信号 USR2: 启动新的主进程，实现热升级
[root@ESB-Main nginx-1.20.1]# kill -USR2 `cat /usr/local/nginx/logs/nginx.pid`
```

### 查看nginx.pid，会出现一个nginx.pid.oldbin
```bash
[root@ESB-Main nginx-1.20.1]# ll /usr/local/nginx/logs
```
![旧文件查看](/img/blog/20210702-4-旧文件查看.jpg)

### 关闭旧的nginx进程(nginx不会停止服务)
```bash
# WINCH: 逐步关闭工作进程
[root@ESB-Main nginx-1.20.1]# kill -WINCH `cat /usr/local/nginx/logs/nginx.pid.oldbin`
```
### 不重载配置启动旧的工作进程
```bash
# HUP: 重新加载配置文件，使用新的配置启动工作进程，并逐步关闭旧进程。
[root@ESB-Main nginx-1.20.1]# kill -HUP `cat /usr/local/nginx/logs/nginx.pid.oldbin`
```
### 结束工作进程，完成此次升级
```bash
# QUIT: 等待请求处理结束后再退出
[root@ESB-Main nginx-1.20.1]# kill -QUIT `cat /usr/local/nginx/logs/nginx.pid.oldbin`
```
### 再次验证
```bash
[root@ESB-Main nginx-1.20.1]# /usr/local/nginx/sbin/nginx -V
```
可以看到nginx版本已经升级到`1.20.1`
![再次验证nginx版本](/img/blog/20210702-5-再次验证nginx版本.jpg)
## 五、总结
至此，nginx成功升级。亲测**`整个过程中nginx代理服务不会中止`**，但还是建议生产系统还是在用户使用频次较低的时间段去升级，避免意外情况的发生。
## 参考

[*查看nginx版本号的几种方法*](https://blog.csdn.net/leiwuhen92/article/details/104405674)
[*nginx 的平滑升级*](https://zhuanlan.zhihu.com/p/193078620)
[*nginx官网下载列表*](https://nginx.org/en/download.html)
[*linux下常用中间件搭建 - nginx搭建(六)*](/2019/07/08/linux下常用中间件搭建-nginx搭建-六/)

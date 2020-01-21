---
title: 堡垒机切换root用户免密配置
categories: [原创, 笔记]
toc: true
date: 2020-01-20 15:39:03
tags: [linux]
---
最近在做系统的迁移，迁移后的服务器全部由堡垒机进行访问，因此配置堡垒机用户切换至root用户免密十分必要。
<!--more-->

## 步骤
```bash
# 1. 堡垒机打开后再次使用输入密码的方式登录root用户
su - root
# 2. 将堡垒机用户添加到wheel用户组，需root用户执行否则无权限，我的堡垒机账号为subadmin
usermod -aG wheel subadmin
# 3. 修改配置
visudo
# 找到这一行，并将注释去掉，保存退出
%wheel        ALL=(ALL)       NOPASSWD: ALL
# 4. 打开新的终端，已经可以免密切换啦
sudo su
```

## 其他用户配置失败情况

若存在配置其他用户（非root）单点登录失败的情况，可以查看目标主机的日志。`tail -200f /var/log/secure`  
若出现的日志信息类似以下错误(我这里新建了app用户)
> Authentication refused: bad ownership or modes for file /home/app/.ssh/authorized_keys 

那么可以进行配置（需要root用户）
```bash
chmod g-w /home/app
chmod 700 /home/app/.ssh
chmod 600 /home/app/.ssh/authorized_keys
# 重启ssh
service sshd restart
# 重启后再试下应该没问题
```

## 相关链接
[解决SSH免密登录配置成功后不生效问题](https://blog.csdn.net/lisongjia123/article/details/78513244)
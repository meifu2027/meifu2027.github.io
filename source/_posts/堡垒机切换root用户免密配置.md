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
---
title: linux服务器之间配置免密登录
categories: [原创, 笔记]
toc: true
date: 2020-01-09 10:53:03
tags: [linux]
---
最近在做服务器搭建的时候，总是需要再多台服务器之间频繁发送文件，因此免密是十分必要的。仅此记录。
<!--more-->
* 生成ssh key(默认生成路径是 /home/$USER/.ssh)
> `ssh-keygen -t  rsa`

![nologin.png](/img/blog/nologin.png)


把A服务器的通过上述指令生成的`id_rsa.pub` 生成的key 配置到B服务器的`authorized_keys`(该文件的目录一般也是`/home/$USER/.ssh`)文件中去，就可以实现免密登录啦。  

------
**注：若服务器一开始设置了不允许root用户登录，则需要修改一下root用户登录的权限**
```bash
vim /etc/ssh/sshd_config
# 修改选项 
PermitRootLogin yes
# 保存退出后重启sshd
service sshd restart
```

**是不是很简单呢，你会了吗？**

## 参考链接
[linux远程登录ssh免密码](https://blog.csdn.net/zhuying_linux/article/details/7049078)
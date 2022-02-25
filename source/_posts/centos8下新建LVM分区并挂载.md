---
title: centos8下新建LVM分区并挂载
categories: [原创, 笔记]
toc: true
date: 2020-01-20 14:56:29
tags: [linux, LVM]
---
最近在迁移系统，网管分配好存储之后并未进行分区格式化并进行挂载，故自己操作以作记录。
<!--more-->
## 步骤
```bash
# 1. 查看磁盘名
fdisk -l

# 2. 磁盘格式化(我这里要操作的磁盘是/dev/sda)
fdisk /dev/sda
# 根据提示一次输入以下指令
# 注："31" 在centos7中应该是 "8e"
m n p t 31 w
# 再次查看（我这边查到新增的是 /dev/sda4）
fdisk -l

# 3. 创建pv卷
# 先查看，应该是暂时没有 /dev/sda4的
pvs
# 创建新的pv卷，创建好之后可以再次查看，应该有了
pvcreate /dev/sda4

# 4. 创建新的vg卷或扩容
# 先查看，应该没有新的
vgs
# 创建新的vg卷，我取名vg卷的名称为data
vgcreate data /dev/sda4
# 若存在扩容情况，则直接在已有vg卷上扩容，假设扩容的分区为/dev/sda5
vgextend data /dev/sda5
# 再次查看应该已经有了

# 5. lvdisplay指定data的vg卷创建lvm卷(我这里命名为datafile，若vgs查出来120G，则可配置119.99G)
lvcreate -L 119.99G -n datafile data
# 若是扩容的情况则是
lvextend -L +119.99G -n datafile data

# 6. 格式化文件系统
# 使用mkfs.ext4命令在逻辑卷data上创建ext4文件系统
mkfs.ext4 /dev/data/datafile

# 7. 挂载到目录/u01
# 注：lvm卷管理的磁盘，无法用mount /dev/sda4 /u01挂载，会报错
mount /dev/data/datafile /u01

# 8. 设置开机启动
echo "/dev/data/datafile /u01 ext4 defaults 0 0" >> /etc/fstab

# 9. 重启验证，重启后查看是否正常挂载即可
reboot

```
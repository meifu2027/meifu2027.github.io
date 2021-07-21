---
title: Linux不重启识别新挂载磁盘
categories:
  - 原创
toc: true
date: 2021-07-21 11:48:16
tags: [linux]
---
> 项目组之前申请的磁盘扩容通过了，但是通过`fdisk -l`命令并没有看到。经查询需要重启或者扫描来看到新挂载的磁盘，因此记录下。  
<!-- more -->

## 步骤
### 未扫描前查看磁盘 
`fdisk -l`
![未扫描前](/img/blog/20210721-1-未扫描前.jpg)
### 查看主机总线号 
`ls /sys/class/scsi_host/`
![查看磁盘](/img/blog/20210721-2-查看磁盘.jpg)
### 重新扫描SCSI总线
` echo "- - -" > /sys/class/scsi_host/host0/scan`
` echo "- - -" > /sys/class/scsi_host/host1/scan`
` echo "- - -" > /sys/class/scsi_host/host2/scan`
![扫描磁盘](/img/blog/20210721-3-扫描磁盘.jpg)
### 再次查看 
`fdisk -l`
![重新查看新挂载磁盘](/img/blog/20210721-4-重新查看新挂载磁盘.jpg)

## 参考
[*Linux不重启识别新挂载的磁盘*](https://www.huaweicloud.com/articles/80e51c71718a7d2e19bcf320a11fd1ae.html)

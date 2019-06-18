---
title: IDEA 同一项目多版本、多窗口打开配置启动配置共享
categories: [原创,笔记]
toc: true
date: 2019-06-18 13:03:04
tags: [idea]
---

> 使用IDEA做java项目开发的朋友可能经历过这样的情况：需要检出多个版本的代码或者需要打开新检出的项目的时候，发现在启动模块的时候需要重新配置。这个时候，做分布式开发的朋友就很痛苦了，是不是所有模块的启动配置都要重新配置呢？答案显然是否定的。
<!--more-->
## 问题描述
项目代码管理服务器搬迁，直接切换地址更新时出现奇奇怪怪的异常问题，所以重新检出来一份最新的代码，用IDEA打开后发现所有的启动配置都需要重新配置（**奔溃！分布式项目，模块多达几十个**）。

## 解决办法
google了一下，看到了可以生效的解决方案，[How do I share IntelliJ Run/Debug configurations between projects?](https://stackoverflow.com/questions/24642147/how-do-i-share-intellij-run-debug-configurations-between-projects?newreg=bd08a870ecca4c4c886fb9c636c1d5f0)
大概的操作是：
1. Run-->Edit Configurations-->勾选需要的启动配置项的"Share"。 ![share1.png](https://i.loli.net/2019/06/18/5d08838c1d0f815656.png)
2. 复制配置文件到新的项目的对应的目录中去![文件路径.png](https://i.loli.net/2019/06/18/5d088338aa26380730.png)

## 总结
IDEA的使用技巧很多，慢慢累计吧，如果有问题的也欢迎留言交流。

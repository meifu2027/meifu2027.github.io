---
title: git检出远程分支
categories: [原创, 笔记]
toc: true
date: 2020-01-10 11:34:43
tags: [git]
---
我们在新克隆一个项目的时候只有master被默认检出，那么其余的分支呢？
<!--more-->
## 检出远程分支
```git
# 查看远程分支
git branch -a
# 将远程分支检出到本地并命名
git checkout -b localBranchName remoteBranchName
# 检查分支
git branch
```

## 参考链接
[git checkout 远程分支](https://blog.csdn.net/ISaiSai/article/details/51350898)
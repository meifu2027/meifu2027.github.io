---
title: linux下常用中间件搭建 - git安装(十)
categories: [原创, 教程]
toc: true
date: 2019-07-10 15:53:34
tags: [linux, git]
---


## 概况
**git版本：** 2.22.0  
**git安装包：** git-2.22.0.tar.gz  
<!--more-->
**其他依赖环境：**  
> zlib-1.2.8.tar.gz（前面章节已安装，可能还会有其他依赖，到时make的时候报错再下载对应的依赖包即可）


**服务器（1台）**  
> A：192.168.0.1  
> B：192.168.0.2   
> 同上一篇当中的maven安装在同一台服务器上。



## 步骤
### 解压、安装


```bash
# 解压
tar -zxvf /u01/setup/git-2.22.0.tar.gz -C /u01
# 编译、安装
cd /u01/git-2.22.0/
./configure
make && make install
# 完成之后查看版本
git --version
```

### 配置SSH Key

一般git安装完之后会生成一组密钥对（公钥、私钥），如果没有的话请查看[官方生成密钥教程](https://git-scm.com/book/zh/v1/%E6%9C%8D%E5%8A%A1%E5%99%A8%E4%B8%8A%E7%9A%84-Git-%E7%94%9F%E6%88%90-SSH-%E5%85%AC%E9%92%A5)
```bash
# 在用户目录下 pub结尾是公钥
cd ~/.ssh
# 查看公钥，复制出来
cat id_rsa.pub
# 在gitlab中配置进去，如图1（后续有时间可以写一个gitlab离线搭建的教程）
```
图1  
![1.png](https://i.loli.net/2019/07/10/5d25a67fee66f64074.png)

### 克隆代码

```bash
# 创建目录
mkdir /u01/.git_project
cd /u01/.git_project
# 因为已经配置了sshkey，因此clone代码时无需输入账号密码
git clone git@192.168.0.1:pms/demo.git
```
git初次clone出来的都是master，一般生产上都会存在开发的分支，因此我们可以检出分支。以下列出几个基本的命令，更详细请移步[git学习入门](https://meifu2027.github.io/2018/05/12/git%E5%AD%A6%E4%B9%A0%E5%85%A5%E9%97%A8/)

```bash
# 查看远程分支
git branch -a 
# 检出上条命令查找到的远程分支：remote_branch_name 本地命名名称：remote_branch_name 
git checkout -b local_branch_name remote_branch_name
# 检查下 本地分支是否创建成功
git branch
# 切换分支到master
git checkout master
```
### maven 打包测试
一般生产的发布都是在master上发布的，所以一般都在master分支上打包。

```bash
# 如果能正常结束，恭喜你打包成功
mvn clean package
```


## 总结
git的离线安装在我这边是比较简单的，但是不同版本的linux可能安装会麻烦一些。因为git安装需要依赖很多安装包，如果一开始就是离线环境，那么所有的依赖包都要自己先下过来安装上，最后才能成功。只能是见招拆招了。

 git装完之后自然是要`下载代码-->mvn打包-->分发到各台服务器指定目录-->启动应用`这么一个过程。等后续涉及一键打包、发布脚本编写的时候再详细介绍。

附上[git下载集合](https://mirrors.edge.kernel.org/pub/software/scm/git/)


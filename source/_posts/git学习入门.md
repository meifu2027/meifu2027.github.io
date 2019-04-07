---
title: git学习入门
date: 2018-05-12 17:51:43
tags: [git, 版本管理]
categories: [教学,原创]
toc: true
---
> 前段时间学习了廖雪峰老师的Python教程，学完后准备跟着一起做实战，发现第一步便是使用GitHub。注册是挺久了，由于项目中用的都是SVN，也没去了解Git和GitHub的使用，趁这个机会记录下Git学习的入门知识点。学习地址，[Git教程](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000)，[Python教程](https://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000)。  

<!--more-->
## 安装Git
### Linux安装Git
试着输入`git`，看看系统有没有安装Git，如果没有安装：
> Debian或Ubuntu Linux:
>  `sudo apt-get install git`
>  老一点的Debian或Ubuntu Linux，命令改为：
>  `sudo apt-get install git-core`
> 其他Linux版本，可直接通过源码安装。官网下载-->解压-->依次输入：`./config`,`make`,`sudo make install`这几个安装命令
### Mac OS X 上安装Git
* 一、通过homebrew安装Git；
* 二、通过安装Xcode安装Git；
### Windows上安装Git
直接从官网下载安装程序，然后默认选项安装即可；
安装完成后，需要设置：
> git config --global user.name "`Your Name`"
> git config --global user.email "`email@example.com`"
## 创建版本库
* 创建空目录：

```bash
$ mkdir learngit 
$ cd learngit
$ pwd
/Users/michael/learngit
```
* 通过`git init`把目录变成Git可以管理的仓库；（文件夹下会多了一个.git的隐藏文件）
* git添加分为两步：
	1. 使用命令`git add <file>`，注意，可反复多次使用，添加多个文件；
	2. 使用命令`git commit -m "<comment>"`，完成。

 

## 时光机穿梭
> `git status` 查看工作区状态；
> `git diff` 查看修改内容；
### 版本回退
* HEAD指向的版本就是当前版本，因此，Git允许我们在版本的历史之间穿梭，使用命令:
> 根据提交版本号回退：`git reset --hard commit_id`
> 提交到上一个版本：`git reset --hard HEAD^`
> 提交往前100个版本：`git reset --hard HEAD~100`  

* 穿梭前，用`git log --pretty=oneline`可以查看提交历史，以便确定要回退到哪个版本。

* 要重返未来，用`git reflog`查看命令历史，以便确定要回到未来的哪个版本。
### 工作区和暂存区
> `git add`是先把工作区的所有修改放到暂存区；
> `git commit`是把暂存区的所有分支提交到分支，之后暂存区就没有任何内容了；
### 管理修改
对同一个文件进行多次修改提交两种方式：
1. 第一次修改-->第二次修改-->`git add`-->`git commit`
2. 第一次修改-->`git add`-->第二次修改-->`git add`-->`git commit`

### 撤销修改
1.  工作区修改，但并未提交到暂存区的，用命令`git checkout -- file`可以还原；
2. 工作区已经提交至暂存区，需要先`git reset HEAD file`，然后操作步骤1；
3. 已经提交到分支上了并且没有推送到远程库的，使用版本回退命令；
### 删除文件
* `git rm file`删除本地文件，并且`git commit` 就会删掉版本库；
* 本地若是错删，可以使用`git checkout -- file`恢复（只能恢复最新的，本地的修改都不会被恢复）；


## 远程仓库

1. 创建SSH Key。在用户主目录下，看看有没有.ssh目录，如果有，再看看这个目录下有没有`id_rsa`和`id_rsa.pub`这两个文件，如果已经有了，可直接跳到下一步。如果没有，打开Shell（Windows下打开Git Bash），创建SSH Key：
> `ssh-keygen -t rsa -C "youremail@example.com"`
其中，`id_rsa`是私钥，`id_rsa.pub`是公钥;

2.  登陆GitHub，打开“Account settings”，“SSH Keys”页面：然后，点“Add SSH Key”，填上任意Title，在Key文本框里粘贴`id_rsa.pub`文件的内容;
### 添加远程库
登陆GitHub找到`Create a new repo`,在`Repository name`填入`learngit`，其他保持默认设置，点击`Create repository`按钮，就成功地创建了一个新的Git仓库;
根据GitHub的提示，在本地的`learngit`仓库下运行命令：
> 远程库的名字默认叫做`origin`
> `git remote add origin git@github.com:michaelliao/learngit.git`
> 把本地库的所有内容推送到远程库上：
> `git push -u origin master`

由于远程库是空的，我们第一次推送`master`分支时，加上了`-u`参数，Git不但会把本地的`master`分支内容推送的远程新的`master`分支，还会把本地的`master`分支和远程的`master`分支关联起来，在以后的推送或者拉取时就可以简化命令。
从现在起，只要本地作了提交，就可以通过命令：
> 把本地master分支的最新修改推送至GitHub
> `git push origin master`
### 从远程库克隆
> `git clone git@github.com:michaelliao/gitskills.git`
> Git支持多种协议，包括`https`，但通过`ssh`支持的原生`git`协议速度最快
## 分支管理
### 创建与合并分支
创建并切换到`dev`分支:
> `git checkout -b dev`  

 相当于：
> `git branch dev` 创建
> `git checkout dev`  切换


查看当前分支：
> `git branch`

 切换`master`分支：
> `git checkout master`

合并`dev`上的代码到`master`上:
> `git merge dev`

删除`dev`分支:
> `git branch -d dev`
### 解决冲突
当`git merge feature1`命令展示冲突信息，需要手动去合并冲突信息；
合并后再`git add`,`git commit`;
查看分支合并情况：
> `git log --graph --pretty=oneline --abbrev-commit`

### 分支管理策略
合并分支使用：
> `git merge --no-ff -m "merge with no-ff" dev`
###Bug分支
若当前在`dev`分支上开发，但是需要修复`master`上的缺陷，那么需要：
储藏当前开发现场：
> `git stash`
> 
切换`master`分支：
> `git checkout master`

创建`bug-101`分支：
> `git checkout -b bug-101`

修复完成后切回`master`，合并`bug-101`，最后并删除`bug-101`分支:
> `git checkout master `
> `git merge --no-ff -m "fixed bug-101" bug-101`
> `git branch -d bug-101`

切回`dev`开发并查看前面保存的现场:
> `git checkout dev`
> `git stash list`

恢复现场：
> 方式1：恢复和删除分开（当然也可以不删除stash）
> `git stash apply`
> `git stash drop`
> 方式2：恢复的同时删除stash
> `git stash pop`

可以多次`stash`，恢复的时候，先用`git stash list`查看，然后恢复指定的`stash`，用命令：
> `git stash apply stash@{0}`
###Feature分支
如果要丢弃一个没有被合并过的分支，可以强行删除:
> `git branch -D <name>`
###多人协作
查看远程库的信息：
> `git remote`
> 或展示更详细信息：
> `git remote -v`

推送本地分支到远程库：
> `git push origin master`
> `git push origin dev`

分支是否推送:



* `master分支`是主分支，因此要时刻与远程同步；
* `dev分支`是开发分支，团队所有成员都需要在上面工作，所以也需要与远程同步；

* `bug分支`只用于在本地修复bug，就没必要推到远程了，除非老板要看看你每周到底修复了几个bug；

* `feature分支`是否推到远程，取决于你是否和你的小伙伴合作在上面开发。

抓取分支：
> 克隆目录：
>` git clone git@github.com:michaelliao/learngit.git`
> 创建目录：
> `git checkout -b dev origin/dev`
> 推送目录：
> `git push origin dev`
> 若有冲突,拉取到本地解决冲突再推送：
> `git pull`

若`git pull`也失败了，原因是没有指定本地`dev分支`与远程`origin/dev分支`的链接，根据提示，设置`dev`和`origin/dev`的链接：
> `git branch --set-upstream dev origin/dev`

小结：

* 查看远程库信息，使用`git remote -v`；
本地新建的分支如果不推送到远程，对其他人就是不可见的；

* 从本地推送分支，使用`git push origin branch-name`，如果推送失败，先用`git pull`抓取远程的新提交；

* 在本地创建和远程分支对应的分支，使用`git checkout -b branch-name origin/branch-name`，本地和远程分支的名称最好一致；

* 建立本地分支和远程分支的关联，使用`git branch --set-upstream branch-name origin/branch-name`；

* 从远程抓取分支，使用`git pull`，如果有冲突，要先处理冲突。
## 标签管理
###创建标签
切换到分支：
> `git checkout master`

给分支打上标签：
> `git tag v1.0`

查看所有标签：
> 按照字面排序，非时间排序
> `git tag`

给之前提交的版本打分支：
> 第一步，先找到提交的版本号：
> `git log --pretty=oneline --abbrev-commit`
> 第二步，指定版本号打上标签：
> `git tag v0.9 6224937`

查看标签信息：
> `git show <tagname>`

创建带有说明的标签:
> 用`-a`指定标签名，`-m`指定说明文字
> `git tag -a v0.1 -m "version 0.1 released" 3628164`

###操作标签
删除标签：
> `git tag -d v0.1`

推送标签到远程:
> `git push origin <tagname>`

一次性推送全部尚未推送到远程的本地标签:
> `git push origin --tags`

删除已推送远程标签：
> 第一步，删除本地：
> `git tag -d v0.9`
> 第二步，从远程删除：
> `git push origin :refs/tags/v0.9`





## 使用GitHub
* 在GitHub上，可以任意Fork开源仓库；
* 自己拥有Fork后的仓库的读写权限；
* 可以推送pull request给官方仓库来贡献代码
## 使用码云
暂不使用，先不记录；
## 自定义Git
暂不使用，先不记录；




##附录
[Git Cheat Sheet](https://pan.baidu.com/s/1kU5OCOB#list/path=/pub/git)
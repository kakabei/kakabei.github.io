---
layout: post
title: 常用命令 git 
date: 2024-04-01 19:21:12
categories: 工具
tags: 工作提效
excerpt: Frequently Used Command Reference git
---

# 撤销修改

#### 一、未使用 git add 缓存代码

撤销指定文件： 

```sh
git checkout -- filepathname
``` 

撤销所有的文件修改可以使用 

```sh
git checkout .
```

对新建文件无效

####  二、已经使用了  git add 缓存了代码

撤销指定文件:  

```sh
git reset HEAD filepathname
```

撤销所有的文件修改可以使用  

```sh 
git reset HEAD .
``` 

清除 git  对于文件修改的缓存。相当于撤销 git add 命令所在的工作。

####  三、已经用 git commit  提交了代码

来回退到上一次commit的状态 :

```sh 
git reset --hard HEAD^ 
```

此命令可以用来回退到任意版本：

```sh 
git reset --hard  commitid

```

# 分支

```sh
git fetch --prune  # 刷新本地和远程一样

git branch -d 分支名  #  删除本地分支

git checkout -b fix-instance-count origin/main  # 在本地创建一个分支，同时把远程的分支同步下来

git push -u origin new-branch   # 把分支同步到远程仓库
```

git 提交一个分支，发现远程已经被删除，如何再次提交可以用 ：

```sh
git push origin develop_2.0.4
```

[Git 撤销合并——如何在 Git 中恢复之前的合并提交](https://www.freecodecamp.org/chinese/news/git-undo-merge-how-to-revert-the-last-merge-commit-in-git/)


创建切换分支：

```sh 
git ckeckout  -b feature/1.15.1-kane
```
提交分支

```sh 
git push origin feature/1.15.1-kane 
``` 


基于 Merge Request 的开发流程 : https://wikinote.gitbook.io/git-learning/gitlab-cao-zuo/gitlab-merge-request
如何撤销 Merge Request？https://wikinote.gitbook.io/git-learning/gitlab-cao-zuo/gitlab-merge-request-revert
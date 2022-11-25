---
layout: post
title: git 撤销修改
date: 2017-05-20 16:12:15
categories: tools
tags: git 工作经验
excerpt: git 在 add、commit 之后如何撤销修改 
---

# 一、未使用 git add 缓存代码

撤销指定文件： `git checkout -- filepathname` 

撤销所有的文件修改可以使用： `git checkout .`

对新建文件无效

# 二、已经使用了  git add 缓存了代码

撤销指定文件:  `git reset HEAD filepathname`

撤销所有的文件修改可以使用：  `git reset HEAD .` 

清除 git  对于文件修改的缓存。相当于撤销 git add 命令所在的工作。

# 三、已经用 git commit  提交了代码

来回退到上一次commit的状态： `git reset --hard HEAD^ `

此命令可以用来回退到任意版本：`git reset --hard  commitid`
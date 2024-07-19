---
layout: post
title: 常用命令 git 
date: 2024-04-01 19:21:12
categories: 工具
tags: 工作提效
excerpt: Frequently Used Command Reference git
---


# 全局配置

```sh
git config --gobal core.autocrlf false
git config --global user.name "carlos"
git config --global user.email "1447675994@qq.com"

git init   // 初始化一个仓库

sudo git config --system alias. st status
sudo git config --system alias. ci commit
sudo git config --system alias. co checkout
sudo git config --system alias. br branch

git config --global alias. st status
git config --global alias. ci commit
git config --global alias. co checkout
git config --global alias. br branch
git config --global color. ui true
```

# 常用操作

```sh

git add  .            # 添加到缓冲区 可以加 参加 -A
git status            #  查看当前状态
git commit -m "XXX"   # 提交代码
git diff              #  查看不同
git log               # 查看日志
git log --pretty=oneline
git reset --hard HEAD^
git rm # 文件名(包括路径) 从git中删除指定文件  # 删除文件

git reflog # 记录你的每一次命令

git checkout -- file. # 可以丢弃工作区的修改  命令中的--很重要，没有--，就变成了“切换到另一个分支”的命令
git reset HEAD file. # 可以把暂存区的修改撤销掉

git remote add origin git@server-name:path/repo-name.git # 要关联一个远程库；
git push -u origin master  # 使用命令第一次推送master分支的所有内容

git clone #  克隆一个本地库：
git clone git@github.com:michaelliao/gitskills.git
git checkout -b dev # 创建dev分支，然后切换到dev分支

git branch  #命令会列出所有分支，当前分支前面会标一个-号
git merge dev  # 把dev分支的工作成果合并到master分支上
git branch -d dev  # 删除dev分支了
git merge --no-ff -m "merge with no-ff" dev  # 合并分支代码 --no-ff参数，表示禁用Fast forward 这样，从分支历史上就可以看出分支信息。

git stash  # Git还提供了一个stash功能，可以把当前工作现场“储藏”起来，等以后恢复现场后继续工作
git stash apply # 恢复，但是恢复后，stash内容并不删除
git stash drop #来删除
git stash pop，#恢复的同时把stash内容也删了
git branch -D feature-vulcan  #强行删除
 
```
 
 
# 本地暂存 git stash

git 把 stash 内容存在某个地方了，但是需要恢复一下，有两个办法
- 一是用git stash apply 恢复，但是恢复后，stash 内容并不删除，你需要用 git stash drop 来删除；
- 另一种方式是用 git stash pop，恢复的同时把 stash 内容也删了

```sh
git stash list [<options>]
git stash show [<stash>]
git stash drop [-q|--quiet] [<stash>]
git stash ( pop | apply ) [--index] [-q|--quiet] [<stash>]
git stash branch <branchname> [<stash>]
git stash save [-p|--patch] [-k|--[no-]keep-index] [-q|--quiet]
         [-u|--include-untracked] [-a|--all] [<message>]
git stash [push [-p|--patch] [-k|--[no-]keep-index] [-q|--quiet]
         [-u|--include-untracked] [-a|--all] [-m|--message <message>]]
         [--] [<pathspec>…​]]
git stash clear
git stash create [<message>]
git stash store [-m|--message <message>] [-q|--quiet] <commit>git_stash.html
```
# 标签

```sh
git tag <name>  # 就可以打一个新标签
git tag      # 查看所有标签
git tag v0.9 6224937   #v0.9标签名　6224937　commit id
git show <tagname>　 # 查看标签信息　
git tag -d <tagname>　# 删除标签
git push origin <tagname> # 可以推送一个本地标签；
git push origin --tags # 可以推送全部未推送过的本地标签；
git tag -d <tagname> # 可以删除一个本地标签；
git push origin :refs/tags/<tagname> # 可以删除一个远程标签。
```

# 分支

Git鼓励大量使用分支：
```sh
git branch       # 查看分支
git branch <name>  # 创建分支
git checkout <name> # 切换分支
git checkout -b <name>  # 创建+切换分支
git merge <name>   # 合并某分支到当前分支
git branch -d <name> # 删除分支
git branch -v # 查看分支 
git branch A1.0 # 查看分支 
git checkout A1.0 # 切换分支
git branch -d A1.0 # 删除分支   // 如果该分支没有合并到主分支会报错
git branch -D A1.0 # 强制删除

git checkout B  
git merge A   # 把A合并到B中 

git fetch --prune #  刷新本地和远程一样

git push origin feature/1.15.1-kane   // 提交分支
git ckeckout  -b feature/1.15.1-kane # 创建切换分支：
```


git 提交一个分支，发现远程已经被删除，如何再次提交
可以用 ：git push origin develop_2.0.4

[Git 撤销合并——如何在 Git 中恢复之前的合并提交](https://www.freecodecamp.org/chinese/news/git-undo-merge-how-to-revert-the-last-merge-commit-in-git/)
# 分支合并

 比如，如果要将开发中的分支（develop），合并到稳定分支（master），
 
首先切换的master分支：`git checkout master`。

然后执行合并操作：`git merge develop`。

如果有冲突，会提示你，调用git status查看冲突文件。

解决冲突，然后调用 `git add` 或`git rm` 将解决后的文件暂存。

所有冲突解决后，`git commit`  提交更改。

# 忽略文件.gitignore

在Git工作区的根目录下创建一个特殊的.gitignore文件，然后把要忽略的文件名填进去，Git就会自动忽略这些文件
忽略文件的原则是：

- 忽略操作系统自动生成的文件，比如缩略图等；
- 忽略编译生成的中间文件、可执行文件等，也就是如果一个文件是通过另一个文件自动生成的，那自动生成的文件就没必要放进版本库，比如Java编译产生的.class文件；
- 忽略你自己的带有敏感信息的配置文件，比如存放口令的配置文件。

不需要从头写.gitignore文件，GitHub已经为我们准备了各种配置文件，只需要组合一下就可以使用了。所有配置文件可以直接在线浏览：https://github.com/github/gitignore

这个仓库要忽略的文件

   - 忽略所有 .a 结尾的文件　-.a
  - 但 lib.a 除外 !lib.a
   - 仅仅忽略项目根目录下的 TODO 文件，不包括 subdir/TODO  /TODO
   - 忽略 build/ 目录下的所有文件 build/
   - 会忽略 doc/notes.txt 但不包括 doc/server/arch.txt doc/-.txt

```sh
    .project
    .pydevproject
    .settings/
    -.ncb
    -.suo
    -.user
    .cproject
    -.o
```

# 撤销修改

## 未使用 git add 缓存代码

撤销指定文件： `git checkout -- filepathname` 
撤销所有的文件修改可以使用 `git checkout .`
对新建文件无效

##  已经使用了  git add 缓存了代码

撤销指定文件:  `git reset HEAD filepathname`
撤销所有的文件修改可以使用  `git reset HEAD .` 
清除 git  对于文件修改的缓存。相当于撤销 `git add` 命令所在的工作。

## 已经用 git commit  提交了代码

来回退到上一次commit的状态 `git reset --hard HEAD^ `

此命令可以用来回退到任意版本：`git reset --hard  commitid`




基于 Merge Request 的开发流程 : https://wikinote.gitbook.io/git-learning/gitlab-cao-zuo/gitlab-merge-request
如何撤销 Merge Request？https://wikinote.gitbook.io/git-learning/gitlab-cao-zuo/gitlab-merge-request-revert

# .gitconfig 配置

```sh
[core]
        repositoryformatversion = 0
        filemode=true
        bare=false
        logallrefupdates=true
        symlinks=false
        ignorecase=true
        hideDotFiles=dotGitOnly
        autocrlf=true
        quotepath=false

[remote "origin"]
        url = https://github.com/alonegarden/alonegarden.github.io.git
        fetch = +refs/heads/*:refs/remotes/origin/*
[branch "master"]
        remote = origin
        merge = refs/heads/master
[user]
        name=cxxlxx
        email=xxxxxxxxxx@qq.com
[color]
        ui=true
[alias]
        co=checkout
        ci=commit
        br=branch
        st=status
        last=log -1
        lg="log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr)%Creset' --abbrev-commit --date=relative"
        lgo="log origin/master --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr)%Creset' --abbrev-commit --date=relative"
```


# 遇到的问题

`git add .` 之后出现  ：
```sh 
The file will have its original line endings in your working directory
```

要设置一下 `git config --global core.autocrlf false`

# github

协议： [github常见开源协议概括](https://cloud.tencent.com/developer/article/1921909)

表情：[https://github.com/zhouie/markdown-emoji](https://github.com/zhouie/markdown-emoji)
数据牌是一种轻量级的徽章:  [https://shields.io/](https://shields.io/)

<p align="center">
<img src="https://github.com/crispgm/resume/workflows/build/badge.svg" alt="GitHub CI" />
<a href="https://badge.fury.io/rb/jekyll-theme-minimal-resume">
<img src="https://badge.fury.io/rb/jekyll-theme-minimal-resume.svg" alt="Gem Version" />
</a>
<a href="https://badge.fury.io/js/hexo-theme-crisp-minimal-resume">
<img src="https://badge.fury.io/js/hexo-theme-crisp-minimal-resume.svg" alt="npm version" />
</a>
</p>

```html
<p align="center">
<img src="https://github.com/crispgm/resume/workflows/build/badge.svg" alt="GitHub CI" />
<a href="https://badge.fury.io/rb/jekyll-theme-minimal-resume">
<img src="https://badge.fury.io/rb/jekyll-theme-minimal-resume.svg" alt="Gem Version" />
</a>
<a href="https://badge.fury.io/js/hexo-theme-crisp-minimal-resume">
<img src="https://badge.fury.io/js/hexo-theme-crisp-minimal-resume.svg" alt="npm version" />
</a>
</p>
```
例子 ： https://github.com/kakabei/jekyll-resume

# 多帐号配置

由于我同时使用了github和码云。所以必须同时配置两个git帐号。
我的所在的环境是windows. （想必linux下也是如此）

```sh
$ cd C:\Users\xxxx\.ssh

$ ssh-keygen -t rsa -C "youremail@xx.com"  

Generating public/private rsa key pair.
Enter file in which to save the key (/c/Users/xxxx/.ssh/id_rsa): ym
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in ym.
Your public key has been saved in ym.pub.

```
要在 `Enter file in which to save the key (/c/Users/xxxx/.ssh/id_rsa)` 后输入你的文件名。
在`C:/Users/xxxx/.ssh`下：(xxxx为用户名)
创建一个config文件，（注意没有后缀）

```sh
Host github.com  
    HostName github.com  
    PreferredAuthentications publickey  
    IdentityFile ~/.ssh/id_rsa  
  
Host git.oschina.net 
    HostName git.oschina.net  
    PreferredAuthentications publickey  
    IdentityFile ~/.ssh/ym 
```
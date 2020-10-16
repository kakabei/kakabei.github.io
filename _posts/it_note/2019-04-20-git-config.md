---
layout: post
title: git config
date: 2017-03-20 16:12:15
categories: tools
tags: git
excerpt: git configure set alias 
---

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
        name=carlos
        email=1447675994@qq.com
[color]
        ui=true
[alias]
        co=checkout
        ci=commit
        br=branch
        st=status
        last=log -1
        lg="log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"
        lgo="log origin/master --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"
```

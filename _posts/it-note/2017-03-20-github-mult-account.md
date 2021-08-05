---
layout: post
title: 多个git帐号的提交
date: 2017-03-20 16:12:15
categories: 七七八八
tags: git
excerpt: 同时在使用多个github帐号的情况下，如何配置。
---

由于我同时使用了github和云码。所以必须同时配置两个git帐号。

我的所在的环境是windows. （想必linux下也是如此）

先cd 到C:\Users\xxxx\.ssh下。

```sh
$ ssh-keygen -t rsa -C "youremail@xx.com"  

Generating public/private rsa key pair.
Enter file in which to save the key (/c/Users/xxxx/.ssh/id_rsa): ym
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in ym.
Your public key has been saved in ym.pub.

```
要在Enter file in which to save the key (/c/Users/xxxx/.ssh/id_rsa)后输入你的文件名。


在C:\Users\xxxx\.ssh下：(xxxx为用户名)

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
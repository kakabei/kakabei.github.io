---
layout: post
title: git cofing 命令
date: 2017-05-21 16:12:15
categories: tools
tags: git 工作经验
excerpt: git 的全局配置和局部配置
---

1.查看 Git 所有配置

```sh
git config --list
```

2.删除全局配置项

终端执行命令：

```sh
git config --global --unset user.name
```

编辑配置文件：

```sh 
git config --global --edit
```
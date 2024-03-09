---
layout: post
title: jekyll blog 的 tags categories 跳转失败
date: 2023-02-17 10:12:15
categories: tools  
tags:   bug 工作提效
excerpt: Jekyll bolg 的问题，更新特征 A，导致问题很久的地方出现问题B，真的很难被发现
---

博客的 tags 和 categories 跳转全部失败了。有点奇怪，之前没有发现这个有什么问题，估计改动了其他的什么功能导致了这个问题吧。

# 问题描述

点击文章中的 category 和 tag 都无法正常跳转，出现如下图情况。

![](/assets/tools/jeklly-2023-02-18-12-17-17.png)

出的错误：

![](/assets/tools/jeklly-2023-02-18-11-58-25.png)

# 解决

感觉是拼接的 url 出现了问题。记得之前在URL后加了`/` 导致无法正常显示出来。 估计也是这个问题，

把 `http://blog.xyecho.com/categories/#c++-ref`  改成 `http://blog.xyecho.com/categories#c++-ref` 也可以正常显示了。

但在本地却没有发现这个问题，应该是本地服务对 URL 没有做严格的匹配吧。

代码发动如下：

![](/assets/tools/jeklly-2023-02-18-12-47-48.png)
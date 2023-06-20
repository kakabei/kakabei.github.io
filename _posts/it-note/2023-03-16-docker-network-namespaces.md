---
layout: post
title: linux network namespaces
date: 2023-03-15 10:12:15
categories: 分布式
tags: docker 
excerpt: Docker network namespaces 
---



一般来说，Linux 系统会安装共享一组网络接口和路由表项。我们可以修改路由表项的策略路由， 但根本上网络接口集和路由表/项还是在在整个操作系统中共享。

使用网络名称空间，您可以拥有相互独立操作的网络接口和路由表的不同且独立的实例。
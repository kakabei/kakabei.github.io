---
layout: post
title: redis 查看版本
date: 2015-02-05 11:08:12
categories: db
tags:  redis
excerpt: redis version
---

查看 redis 的版本, linux 版本。

查看 redis 的版本有两种方式：

### 方式一

```sh 
redis-server --version 
或 
redis-server -v 
```

得到的结果是：

```sh
Redis server v=2.6.10 sha=00000000:0 malloc=jemalloc-3.2.0 bits=32
```

### 方式二

```sh
redis-cli --version 
或
redis-cli -v
```

得到的结果是：`redis-cli 2.6.10`

严格上说：通过　redis-cli 得到的结果应该是redis-cli 的版本，但是 redis-cli 和r edis-server　一般都是从同一套源码编译出的。所以应该是一样的。


---
layout: post
title: redis安装部署
date: 2015-02-03 11:08:12
categories: redis
tags: db redis
excerpt: redis install in centos
---
 

Redis的安装相对来的不难，以下主要是centos系统做为环境。

1. 下载地址

这是redis的官网[http://redis.io/](http://redis.io/)。

这是下载页面[http://redis.io/download](http://redis.io/download)。

百度云地址:[http://pan.baidu.com/s/1slsCr97](http://pan.baidu.com/s/1slsCr97)

2. 安装

```sh
$ tar xzf  redis-2.8.13.tar.gz
$ cd redis-2.8.13
$ make
$ make install
$ cp redis.conf /etc/
```
3. 文件说明

make install命令执行完成后，会在/usr/local/bin目录下生成本个可执行文件，分别是redis-server、redis-cli、redis-benchmark、redis-check-aof 、redis-check-dump，它们的作用如下：

```sh
redis-server：Redis服务器的daemon启动程序
redis-cli：Redis命令行操作工具。也可以用telnet根据其纯文本协议来操作
redis-benchmark：Redis性能测试工具，测试Redis在当前系统下的读写性能
redis-check-aof：数据修复
redis-check-dump：检查导出工具
```

4. 启动redis

```sh
 /usr/local/bin/redis-server /etc/redis.conf
````

5．检查是否启动成功

```sh 
$ ps -ef | grep redis
```

在线测试： [http://try.redis.io/]( http://try.redis.io/)
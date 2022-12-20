---
layout: post
title: mysql 主从原理
date: 2015-05-27 11:08:12
categories: db  
tags: mysql 
excerpt: mysql 主从同步的原理，主要是通过 binlog 的方式实现
---
 
# 一、 主从同步的好处

mysql 主从同步有三个好处：

1、水平扩展数据库的负载能力 。

2、容错，高可用，（Failover / High Availability） 。

3、数据备份。
    
# 二、 主从同步的原理

1、 delete、update、insert，还是创建函数、存储过程，所有的操作都在 master 上。当 master 有操作的时候,slave 会快速的接收到这些操作，从而做同步。

2、mysql 主从同步的原理，主要是通过 binlog 的方式。

- master 提交完事务后，写入 binlog。
- slave 连接到 master，获取 binlog。
- master 创建 dump 线程，推送 binglog 到 slave。
- slave 启动一个 IO 线程读取同步过来的 master 的 binlog，记录到 relay log 中继日志中。
- slave 再开启一个 sql 线程读取relay log 事件并在 slave 执行，完成同步。
- slave 记录自己的 binglog。
  
3、主从同步事件有3种形式: statement、row、mixed。
 
- statement：会将对数据库操作的sql语句写入到 binlog 中。
- row：会将每一条数据的变化写入到 binlog 中。
- mixed：statement 与 row 的混合。
- mysql 决定什么时候写 statement 格式的，什么时候写 row 格式的 binlog。

![](./assets/db/mysql-2022-10-18_22-28-15.png)

# 三、相关线程

1、master 机器 会开启 binlog dump 线程。 当 master 的 binlog 发生变化时， binlog dump 线程会通知 slave 并将相应的 binlog 内容发送给 slave。 

2、slave 机器上，会创建 2 个线程， I/O 线程 和 SQL线程。 

3、I/O 线程，负责连接到 master 机器， 接收 master 机器上的 binlog dump 线程发送过来的内容，再写入本地的 relay log。

4、SQL 线程，读取本地的 relay log, 执行语句写入相应的数据库中。 

  
**疑问：** 

1、是线程，还是进程？ 

是线程。

2、master 和 slave 之间的 TCP 连接吗？ 

master 要创建 slave用户，说明应该是 TCP 连接。

3、数据同步过去，是 master 主动，还是 slave 定时查询？

TCP连接，应该是会是 master 主动发现后发送给slave

4、如果 slave 有很多，那么 master 都要写一遍，对性能的影响会不会很大？

异步的试同步灵气不影响正常查询，也不会有太多 slave， 一般两个，分不同机房。

5、主从复制数据耗时大概在多少？ 看数据量和同步时间间隔。


# 四、主从复制的策略

1、同步策略。master 会等待所有的 slave 都回应后才会提交，这个主从的同步的性能会严重的影响。

2、半同步策略。master 至少会等待一个slave 回应后提交。

3、异步策略。master 不用等待 slave 回应就可以提交。 

4、延迟策略。master 要落后于 master 指定的时间。

  
一般都会采用取终一致性，不会要求强一致性，毕竟对性能的影响比较大。但还是要看具体的业务，不同业务有不同的要求。


# 五、配置 mysqld.cnf 

1、 master 配置 mysqld.cnf 
    
```ini
bind-address = 192.168.33.22 #your master

ipserver-id = 1 #在master-slave架构中，每台机器节点都需要有唯一的

server-idlog_bin = /var/log/mysql/mysql-bin.log #开启binlog 
```

2、创建salve 用户

```sh
$ mysql -u root -pPassword:

##创建slave1用户，并指定该用户只能在主机192.168.33.33上登录。

mysql> CREATE USER 'slave1'@'192.168.33.33' IDENTIFIED BY 'slavepass';

Query OK, 0 rows affected (0.00 sec)


##为slave1赋予REPLICATION SLAVE权限。

mysql> GRANT REPLICATION SLAVE ON *.* TO 'slave1'@'192.168.33.33';

Query OK, 0 rows affected (0.00 sec)
```

3、导数据 即把已经在master上的数据先导到slave 上  

4、salve 上的配置  ：

```ini
bind-address = 192.168.33.33 #your slave ip

server-idlog_bin = /var/log/mysql/mysql-bin.log #开启binlog

```
  
5、创建 slave 和 master 的连接 
``
```
mysql -u root -pPassword:

mysql> STOP SLAVE;
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql> CHANGE MASTER TO

-> MASTER_HOST='192.168.33.22',
-> MASTER_USER='slave1',
-> MASTER_PASSWORD='slavepass',
-> MASTER_LOG_FILE='mysql-bin.000001',
-> MASTER_LOG_POS=613;Query OK, 0 rows affected, 2 warnings (0.01 sec)

mysql> START SLAVE;
Query OK, 0 rows affected (0.00 sec)
```


**疑问：**

1、mysql 主从有什么优点？为什么要选择主从？

高性能方面：主从复制通过水平扩展的方式，解决了原来单点故障的问题。

实现读写分离，不会因为写操作过长锁表而导致读服务不能进行的问题，提高了服务器的整体性能

可靠性方面：主从在对外提供服务的时候，若是主库挂了，会有通过主从切换。


2、若是主从复制，达到了写性能的瓶颈，你是怎么解决的呢？

根据业务可以在设计上进行解决采用分库分表的形式。

主从复制的过程有数据延迟怎么办？导致 Slave 被读取到的数据并不是最新数据。

对于数据的实时性要求很高，要求强一致性，可以采用同步复制策略。但会影响性能。

对于数据延迟的解决方案没有最好的方案，就看你的业务场景中哪种方案使比较适合的。

---
参考资料：

知乎 : [# Mysql主从同步的实现原理与配置实战](https://zhuanlan.zhihu.com/p/89796383)

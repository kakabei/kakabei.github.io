---
layout: post
title: java 技术栈学习脉络
date: 2021-07-15 16:12:15
categories: 学习笔记
tags: java
excerpt: java 技术栈学习脉络
---

继续整理更新中~

### 一、 java基础

1. Linux常用命令
2. Shell 脚本编程
3. 数据结构
4. 算法
5. 集合
6. IO/NIO
7. 并发编程 
   - 并发基石线程基础
   - synchronized
   - JUC
8. JVM 
9. JDBC 连接池
10. 编码规范

### 二、jave web 略过

JSP、Servlet 、Html、CSS、JavaScript、JQuery、Tomcat

### 三 重构设计

设计原则 

1. 开闭原则总则
2. 依赖倒置原则
3. 接口隔离原则 
4. 单一职责原则 
5. 里氏替换原则 
6. 最少知道原则 
7. 合成复用原则

常用设计模式

- 创建型模式
  - 单例模式
  - 工厂模式
- 结构型模式
  - 代理模式
  - 装饰模式
  - 适配器模式
- 行为型模式
  - 策略模式
  - 命令模式
  - 责任链模式
  - 观察者模式
  - 模板方法模式

重构-改善既有代码的设计

### 三、开源框架 （重点）

1. NET 框架 
   ​    - Netty 
   ​    - HttpClient 
   ​    - Mina
2. MVC框架
   - Sping
   - Structs
3. ORM 框架
   - MyBatis 
   - Hibernate
4. RPC 框架
   - Dubbo 
   - springCloud
   - Thrift
5. 模板引擎
   - Velocity 
   - Freemarker

### 四、数据存储

1. SQL数据库
   - MySQL 
   - Oracle 
   - DB2
   - 分库分表 
     ShardingSphere 
     MyCat 
     TDDL 
2. NoSQL 
   - Redis
   - Hbase
   - MongoDB

### 五、测试技能

1. 单元测试MockSpringTest 
2. 压力测试 JMeter

### 六、中间件

1. Redis 
2. RocketMQ 
3. Zookerper
4. elasticStack
5. etcd
6. kafka
7. memcashed 

### 七、性能优化

1. web 前端性能优化 （略过）
2. 应用服务性能优化
   - 集群
   - 缓存
   - 异步
   - 代码
     - 并发
     - 编程
     - 资源得胜
     - 数据结构
   - JVM
3. 数据存取性能优化
   - SQL优化
   - 索引优化
   - 数据库架构 + 分库分表

### 八、架构技能 

1. 分布式架构
   - keepalive + nginx/lvs 
   - Zooperper
   - RPC
     - Dubbo
     - SpringCloud
   - 服务治理
     - 服务熔断
     - 服务降级
     - 服务限流
     - 服务隔离
   - MQ
     - RockerMQ 
     - kafka
   - 缓存
     - redis 
     - memchached 
   - 分布数据一致性
   - 微服务架构 
   - Docker 
   - 数据库架构
     - 主备架构
     - 主从架构
     - 双主架构 （什么场景会有这个？）

### 九、大数据

1. 数据收集
   - 网络爬虫
   - Flume/Logstash 
2. 数据存储
   - HDFSHive (这个算是存储吗？)
   - Hbase
   - MongoDB
3. 数据检索
   - Elasticsearch
4. 数据处理
   - Hive 
   - Storm 
   - ParkFlink 
5. 数据挖掘
   - 机器学习

### 十、解决方案

1. 技术实践方案
2. 业务实现方案

### 十一、其他技能

1. 开发工具
   - Intellij 
   - IDEA
   - Eclipse 
2. 项目构建
   - MavenGradle
3. 版本控制
   - Git
   - SVN 

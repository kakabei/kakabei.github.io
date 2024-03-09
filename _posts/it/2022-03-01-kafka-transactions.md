---
layout: post
title: Kafka 事务操作
date: 2022-03-01 10:12:15
categories: MQ  
tags: kafka 技术学习笔记
excerpt: Kafka事务可以使应用程序将消费消息、生产消息、提交消费位移当作原子操作来处理
---

kafka 事务中，一个原子操作，根据操作类型可以分为3种情况：

1、producer 发送多条消息组成一个事务， 这些消息需要同时可见或同时不可见。

2、producer 给多个 topic 多个partition 发消息，这些消息也需要能放在一个事务里面，这就形成了一个典型的分布事务。 

3、程序先消费一个 topic，然后做处理再发到另一个 topic，这个 consume-transform-produce 过程需要放到一个事务里面，比如在消息处理或者发送的过程中如果失败了，消费位点也不能提交。 

4、producer 或者 producer 所在的应用可能会挂掉，新的producer启动以后需要知道怎么处理之前未完成的事务 。

幂等性并不能跨多个分区运行，而事务可以弥补这个缺陷。

程序必须提供唯一的transactionalId ,要通过客户端参数显示设置。

transactionalId 与 PID 一一对应，两者之间所不同的是 transactionalId 由用户显示设置，而 PID 是由 kafka内部分配的。

 `properties.put(ProducerConfig.TRANSACTIONAL_ID_CONFIG, “transacetionId”);`


KafkaProducer提供了5个与事务相关的方法，详细如下：

```java
// 初始化事务 
producer.initTransactions(); 
// 开启事务 
producer.beginTransaction(); 
// 消费 - 生产模型 
producer.send(producerRecord); 
// 提交消费位移 
producer.sendOffsetsToTransaction(offsets, "groupId"); 
// 提交事务 
producer.commitTransaction();

// 中止事务 
producer.abortTransaction();
```

实现 Kafka 事务，主要使用到 Broker 端的**事务协调器 (TransactionCoordinator)**。每个 Producer 都会被指定一个特定的 TransactionalCoordinator，用来负责处理其事务，与消费者 Rebalance 时的 GroupCoordinator 作用类似。实现事务的流程如下图所示：

![](/assets/mq/kafka-2023-03-14-12-03-10.png)


-----
1、[Kafka技术知识总结之二——Kafka事务](https://cloud.tencent.com/developer/article/1657503)

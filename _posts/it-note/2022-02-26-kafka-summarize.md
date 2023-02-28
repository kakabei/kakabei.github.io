---
layout: post
title: Kafka 学习
date: 2022-02-26 10:12:15
categories: MQ  
tags: MQ kafka 学习笔记
excerpt: 工作中，我对 kafka 的学习与理解，整理成一个简单大致的概述
---

# MQ 
微服务开发中，经常用到了一个比较重要的组件就是 `MQ`。比较流行的 MQ 有 `RabbitMQ`、`RocketMQ`、`Kafka` 等。

在工作中有使用到 `MQ` 的场景，主要目的就是有：

- **异步通信**
- **解耦**
- **削峰**

**异步通信**： 比如业务处理流程中，我的 sku 状态发生了变化，要通知搜推业务方，而他们的业务接口比较重，我用调用他们接口后耗时比较大。这样会导致我的耗时也比较大，这时我们可以选择 MQ，把消息通过 MQ 发送给他们。我是生产者，只负责发送。发送成功即可往下处理自己的流程了，搜推业务方做了消费者他们收到数据后怎么处理不会影响到我接口的 qps 。

**解耦**： 如我的 sku 状态已经发生了，要通知相关的业务方比较多时，如：搜推、商详、商家、活动等等。这时我们需要调用他们的接口，工作量巨大。即使是并发调用他们的接口，只有一个接口延迟比较大，我的接口就会被拖累。如一个接口出问题了，也可能导致调用其他接口也出现异常。 这个就是把所有相关的业务方接口耦合在一起。 我们可以通过 MQ 中间件，做为生产者把消息发送过去。不同的业务方做为消费者处理数据即可，这样不同业务的就不会相互影响。 

**削峰**： 我的流量突然陡增，qps 突然就大了，这时我的接口可以处理过来了，但相关的业务方的接口不一样可以处理过来。这样可能会导致们的接口报错， 或者机器负载突然过高，产生告警。而使用 MQ 中间件的话， 我把数据发给 MQ，做为消费者的业务方可以根据的自己的处理能力，正常处理。让一部分数据先积压在 MQ 上。（这里也有缓冲的作用）从而起到了削峰的目的，保护好自己的接口。 

# Kafka   

我工作中有用到的是 `Kafka`。 `Kafka` 是一个优秀的分布式消息中间件。

## Kafka 主要设计目标

1、以时间复杂度为 `O(1)` 的方式提供持久化能力，

2、高吞吐率做到单机每秒 `100K` 条消息传输。

3、支持 `Kafka server` 间的消息分区，及分布消费，保证每个 `partition` 内的消息顺序传输。

4、支持离线数据处理和实时数据处理。

5、在线水平扩展。

## Kafka 架构中的一般概念

1、**Producer**：生产者，发送消息的一方。生产者负责创建消息，然后将其发送到 `Kafka` 。

2、**Consumer**：消费者，也就是接受消息的一方。消费者连接到 `Kafka` 上并接收消息，进而进行相应的业务逻辑处理。

3、**Consumer Group**：一个消费者组可以包含一个或多个消费者。使用多分区 + 多消费者方式可以极大提高数据下游的处理速度，同一消费组中的消费者不会重复消费消息，同样的，不同消费组中的消费者消息消息时互不影响。`Kafka` 就是通过消费组的方式来实现消息 P2P 模式和广播模式。

4、**Broker**：服务代理节点。`Broker` 是 `Kafka` 的服务节点，即 `Kafka` 的服务器。

5、**Topic**：`Kafka` 中的消息以 `Topic` 为单位进行划分，生产者将消息发送到特定的 `Topic`，而消费者负责订阅 `Topic` 的消息并进行消费。

6、**Partition**：`Topic` 是一个逻辑的概念，它可以细分为多个分区，每个分区只属于单个主题。同一个主题下不同分区包含的消息是不同的，分区在存储层面可以看作一个可追加的日志（Log）文件，消息在被追加到分区日志文件的时候都会分配一个特定的偏移量（offset）。

7、**Offset**：是消息在分区中的唯一标识，`Kafka` 通过它来保证消息在分区内的**顺序性**，不过 `offset` 并不跨越分区，也就是说，`Kafka 保证的是分区有序性而不是主题有序性`。

8、**Replication**：副本，是` Kafka` 保证数据高可用的方式。`Kafka` 同一 `Partition` 的数据可以在多 `Broker` 上存在多个副本，通常只有主副本对外提供读写服务，当主副本所在 broker 崩溃或发生网络一场，`Kafka` 会在 `Controller` 的管理下会重新选择新的 Leader 副本对外提供读写服务。

9、**Record**：实际写入 `Kafka` 中并可以被读取的消息记录。每个 `record` 包含了 key、value 和 timestamp。

## Kafka 大致的架构图

![](/assets/mq/kafka-mq-2023-02-28-14-19-07.png)

先理解了 Kafka 是一些概念之后，对这个框架图就比较好理解了。

### 生产者 producer 

1、`producer` 先从 `zookeeper` 找到主题的 `partition` 的 `leader`。 

2、`producer` 把数据 push 到这个 leader 的 `broker` 中。

3、`leader` 会把数据写入到本地 log。 

4、`follow` 从 `leader` pull 出数据，写入本地 log 然后回复确认给 `leader`，`follow` 做为备份。 

5、`leader` 收到所有确认之后，就向 `producer` 发送确认回复。

### 消费者 customer 

1、 一个 `customer group` 可以包含多个 `customer`。不同的 `customer group` 互不干扰。

2、同组的 `customer` 可以横向扩展，提高消费能力。

3、同组的 `customer `不能多于这个 `topic` 的 `partition`。如果 `customer` 多了，就会出现 `customer` 空跑的情况。如果 `customer` 少了，就是出现一个 `customer` 连接多个 `partition`。最好的一对一的试。 

4、系统扩展时先扩展 `partition`。 再进行 `Rebalance。`   

### Rebalance

`rebalance` 本质上是一种平衡分配的协议。 规定了一个` consumer group` 下的所有 `consumer` 如何达成一致来分配订阅 `topic` 的每个 `partition`。比如某个 gro`up 下有 20 个 consumer，它订阅了一个具有 100 个 `partition` 的 `topic`。正常情况下，`Kafka` 平均会为每个 `consumer` 分配 5 个 `partition`。这个分配的过程就叫 `rebalance`。

**三种情况下如进行 Rebalance**

- 组成员发生变化 
- 订阅主题数发生变化
- 订阅主题的分区数发生变化

# 最后

这个就是 `kafka` 最简单的理解了。先熟悉这大致的流程再一步一步深入的去学习里面的细节和原理。

--- 

1、[https://kafka.apachecn.org/](https://kafka.apachecn.org/)

2、[https://www.cnblogs.com/sodawoods-blogs/p/8969774.html](https://www.cnblogs.com/sodawoods-blogs/p/8969774.html)

3、[https://mp.weixin.qq.com/s/OQrKeFSrNcyTiFXR21pENA](https://mp.weixin.qq.com/s/OQrKeFSrNcyTiFXR21pENA)

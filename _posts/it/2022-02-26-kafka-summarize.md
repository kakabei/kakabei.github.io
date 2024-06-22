---
layout: post
title: Kafka 一种分布式MQ
date: 2022-02-26 10:12:15
categories: MQ  
tags: kafka 技术学习笔记
excerpt: 工作中，我对 kafka 的学习与理解，整理成一个简单大致的概述
---

# MQ

微服务开发中，经常用到了一个比较重要的组件就是 `MQ`。比较流行的 MQ 有 `ActiveMQ`、`RabbitMQ`、`RocketMQ`、`Kafka` 等。

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

# 生产者 Producer

kafka 生产者的流程大致如下图：

![](/assets/mq/kafka-mq-2023-02-17-22-07-19.png)


1、 `Producer` 通过 `RroducerPecord` 封装消息 主要的数据有:topic、partition、key、value、timestamp 等数据。
 
2、通过序列化器把数据序列化，序列化器可以在初始化时指定。 然后再到分区器，由分区器分配到哪个 partition。

3、这个数据被记录到对应的 `topic` 和 `partition` 分类的缓冲区中, 多条消息会被封装成为一个批次（batch），默认一个批次的大小是 16K。 

4、再由独立的线程 Sender 把这些数据发到对应的 `broker` 中。

5、如果服务器（`broker`） 将消息成功写入，就返回 `RecordMetaData` 对象，告诉 `Producer` 客户端主题、分区信息和偏移量。 

6、主题的分区只能增加，不能减少。

7、如果写入失败，则返回错误信息，让 Producer 重试。 

如果没有指明 partition 值，但有 key 的情况下，将 key hash 后与topic 的 partition 数目进行取余操作，得到 partition 值。

工作中，我会把业务一个惟一标识当作 key。像 sku、商家ID 等。 

没有 partition 值又没有 key 的情况下，第一次调用时随机生成一个整数（后面每次调用在这个整数上自增），将这个值与 topic 可用的 partition 总数取余得到 partition 值，也就是 round-robin 算法。


## 分区策略

 kafka 的分区策略有：

 - 轮询策略（Round-robin）
 - 随机策略（Randomness）
 - 按消息键保序策略（Key-ordering）

kafka 也开放性的提供了自定义分区策略 `org.apache.kafka.clients.producer.Partitioner`

```java
public interface Partitioner extends Configurable, Closeable {

    /**
     * Compute the partition for the given record.
     *
     * @param topic The topic name
     * @param key The key to partition on (or null if no key)
     * @param keyBytes The serialized key to partition on( or null if no key)
     * @param value The value to partition on or null
     * @param valueBytes The serialized value to partition on or null
     * @param cluster The current cluster metadata
     */
    public int partition(String topic, Object key, byte[] keyBytes, Object value, byte[] valueBytes, Cluster cluster);

    /**
     * This is called when partitioner is closed.
     */
    public void close();

}

```

### 轮询策略

Round-robin 策略，就是按序分配，这是一种默认的策略。 如果 RroducerPecord 没有 partition 值又没有 key。

假如有四个 partition, 第一个消息分配到 分区 0， 第二消息分配到 分区1 依次类推。到第五个消息又分配到分区0。如下图： 

![](/assets/mq/kafka-mq-2023-02-28-18-05-23.png)

这种策略表现非常好，负载比较均衡。

### 随机策略

Randomness 策略。就是我们随意地将消息放置到任意一个分区上

![](/assets/mq/kafka-mq-2023-02-28-18-16-58.png)

实现随机策略方法: 拿到 topic 的 partition 数，然后用  ThreadLocalRandom 随机出一个比它小的数。

这种方法可能会出不均衡的情况。

```java
List partitions = cluster.partitionsForTopic(topic);
return ThreadLocalRandom.current().nextInt(partitions.size());
```
### 按消息键保序策略

Key-ordering 策略。就是为每一条消息定义一个消息键（key），可以有明确的业务含义，如果 业务编码、部门ID等。 

相同的消息键（key) 会被放到相同的 partition 中。这个策略有一个缺点就是导致数据倾斜。 如下图：

![](/assets/mq/kafka-mq-2023-02-28-18-29-52.png)

代码实现是 用 key hash 之后 对 partition 取模。

```java
List partitions = cluster.partitionsForTopic(topic);
return Math.abs(key.hashCode()) % partitions.size();
```
在分区器上执行策略之后，就会被分发到对应的 partition 上。

##  缓冲区 

 [深度剖析 Kafka Producer 的缓冲池机制](https://cloud.tencent.com/developer/article/1698563)
这个篇文章写的很好，后面自己再分析一下。



# 消息者 Consumer，消费组 Consumer Group

消费者人订阅的主题消费消息，消费消费的偏移量保存 kafka 的名字是 __consumer_offsets 的主题中。

消息者还可以将自己的偏移理存储到 zookeeper， 需要设置 offset.storage=zookeeper.  不适合高并发。

多个从一个主题消费的消费者可以加入到一个消费组中，消费线中的消费者共享group_id。

分区和消费组中的消费者可以一一对应，分区的个数可以多个消费组中的消费者，即有存在一个消费者消费多个分区的情况
如果，分区数少于消费组中的消费者个数，则出现有消费者空跑的情况。如下图：


![](/assets/mq/kafka-2023-03-12-20-33-13.png)

![](/assets/mq/kafka-2023-03-12-20-33-58.png)

消费组与消费组之间是不相互影响的。 


![](/assets/mq/kafka-2023-03-12-20-35-33.png)
消费者负载均衡策略

 一个消费者组中的一个分片对应一个消费者成员，他能保证每个消费者成员都能访问，如果组中成员太多会有空闲的成员。


kafaka 生产数据时数据的分组策略 

生产者决定数据产生到集群的哪个 partition 中 

每一条消息都是以(key，value)格式

Key 是由生产者发送数据传入 所以生产者(key)决定了数据产生到集群的哪个 partition


## 心跳机制

消费者宕机，退出消费组，触发再平衡，重新给消费组中的消费者分配分区。 

![](/assets/mq/kafka-2023-03-12-20-46-48.png)

由于 broker 宕机，主题的分区 3 宕机，此时分区3没有 leader 副本，触发再平衡，消费者4没有对应的主题分区，则消费4闲置。

kafka 的心跳是 kafka consumer 和 broker 之前的健康检查， 只有当broker coordinatio 正常时， consumer 才会发送心跳。 

对应的参数：

broker 端： sessionTimeoutMs   心跳过期时间。 
consumer 端： sessionTimeoutMs, rebalanceTimeouMs  如果客户端发现心跳超期，客户端会coordinator 为不可用，并阻塞心跳线程，如果超过poll 消息的间隔超过了 rebalanceTimeoutMs, 则 consumer 告知 broker 主动离开消费组，也会触发再平衡。 


## 再均衡

再均衡（ rebalance ）本质上是一种协议，规定了一个消费组所有消费者如何达成一致来分配订阅主题的每个分区。 

比如某个消费组有20 个消费组，订阅了一个具有100个分区的主题。正常情况下，kafka 平均会为每个消费分配5个分区。这个分配过程就是再均衡。

如何进行组内分区分配？ 

三种分配策略： RangeAssignor 和 RoundrobinAssignor 以及 StickyAssignor。

谁来执行再均衡和消费组管理

kafka 提供了一个角色： Group Coordinator 来执行对于消费组的管理。

Group Coordinator ：每个消费组分配一个消费组协调器用于组管理和位移管理。当消费组的第一个消费者启动的时候，它会去和kafka broker 确定谁是它们的组的组协调器。之后该消费组所有消费者和该组协调器协调通信。


## 幂等性

**幂等性**： 保证上在消息重发的时候，消费者不会重复处理。即使在消费者收到重复消息的时候，重复处理，也要保证最终结果的一致性。 

kafka 在引入幂等性之前， producer 向 broker 发送消息，然后 broker 将消息追加到消息流的给 producer 返回 ack 信号值。如下图：

![](/assets/mq/kafka-2023-03-14-08-43-08.png)
生产环境中，存在各种不确定因素，比如： producer 在发送给 broker 的时候出现网络异常。比如在回复确认的时候失败了。
![](/assets/mq/kafka-2023-03-14-08-43-34.png)

图中的情况是： 当produce 第一次发送消息给 broker 时， broker 将消息（x2,y2）添加到消息流后返回 ack 确认信息给 producer 时失败了。

此时， producer 端触发生试机制，将消息（x2,y2）再发给 Broker， 再次被追加到消息流中，然后返回 ack 确认消息。 

这时，消息流中就有两条相同的（x2,y2）的消息。

**幂等性实现**

添加唯一ID，类似于数据库的主键，用于唯一票房一个消息。 

kafka 为了实现幂等性，它在底层设计架构中引入了producerID 和 sequenceNumber。
- producerID: 在每个新的producer 初始化时，会被分配一个唯一的 ProducerID, 这个producerID 对客户端使用者是不可见的。
- sequenceNumber : 对于每个 Producer 发㳠数据的每个 topic 和 partiion 都对应一个从0 开始中单调递增的 sequenceNumber 值。

![](/assets/mq/kafka-2023-03-1-08-55-13.png)

同样，这是一个理想状态下的发送流程，实际情况下，会有很多不确定的因素，比如 broker 发送 ack 信息给 producer 时出现网络异常，导致发送失败。异常情况如下图所示：

![](/assets/mq/kafka-2023-03-14-08-56-26.png)

当 producer 发送消息（x2,y2） 给 broker 时，broker 接收消息并将其追加到消息流中。此时， broker 返回 ack 信息给 producer 失败，producer 触发重试机制，将消息（x2,y2）再次发送，但是由于引入了幂等性，在每条消息中附带了PID 和 sequenceNumber。相同的PID 和 sequenceNumber 发送给 broker， broker 会检测是否已经在有重复的，保存不存在相同的记录。  

## 消费者位移管理

kafka 中，消费者根据消息的位移顺序消费消息。 

## 消费者管理

consumer group 是 kafka 提供可扩展且具有容错性的消费者机制。

三个特性：

1、消费组有一个或多个消费者，消费者可以是一个进程，也可以是一个线程。
2、group.id 是一个字符串，唯一标识一个消费组。
3、消费组订阅的主题每个分区只能分配给消费一个消费者。

消费者位移

消费者在消费的过程中记录已消费的数据，即消费位移（offset） 信息。
每个消费组只在自己的位移信息，那么只需要简单的一个整数表示位置就够了；同时可以引入 checkpoint 机制定期持久化。

## 位移管理 

kafka 默认定期自动提交位移，定期把 group 消费情况保存起来，做成一个 offset map 如下图所示：

![](/assets/mq/kafka-2023-03-13-22-31-41.png)

# 事务

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

# 最后

这个就是 `kafka` 最简单的理解了。先熟悉这大致的流程再一步一步深入的去学习里面的细节和原理。

当然，kafka 也存在一些缺点：

- 系统复杂性剖。 kafka 的引入是一个比较复杂的分布式中间件。管理、成本要求都比较高。 
- 数据一致性问题。kafka 解决了自身的一致性问题，但是却可能给业务系统带来数据一致性问题，如：A系统把数据通过 kafka 发给给 B系统。A  B之间的数据可以不一致，出现丢失的情况。一般我的处理方式是兜底程序发现数据不一致时进行兜底处理。 

--- 

参考：

1、[https://kafka.apachecn.org/](https://kafka.apachecn.org/)

2、[https://www.cnblogs.com/sodawoods-blogs/p/8969774.html](https://www.cnblogs.com/sodawoods-blogs/p/8969774.html)

3、[https://mp.weixin.qq.com/s/OQrKeFSrNcyTiFXR21pENA](https://mp.weixin.qq.com/s/OQrKeFSrNcyTiFXR21pENA)

4、	[Kafka技术知识总结之二——Kafka事务](https://cloud.tencent.com/developer/article/1657503)
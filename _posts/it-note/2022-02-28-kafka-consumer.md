---
layout: post
title: Kafka 消息者和消费组
date: 2022-02-28 10:12:15
categories: MQ  
tags: kafka 技术学习笔记
excerpt: Kafka 消息者和消费组来消费消息。
---

# 消息者，消费组

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


# 心跳机制

消费者宕机，退出消费组，触发再平衡，重新给消费组中的消费者分配分区。 

![](/assets/mq/kafka-2023-03-12-20-46-48.png)

由于 broker 宕机，主题的分区 3 宕机，此时分区3没有 leader 副本，触发再平衡，消费者4没有对应的主题分区，则消费4闲置。

kafka 的心跳是 kafka consumer 和 broker 之前的健康检查， 只有当broker coordinatio 正常时， consumer 才会发送心跳。 

对应的参数：

broker 端： sessionTimeoutMs   心跳过期时间。 
consumer 端： sessionTimeoutMs, rebalanceTimeouMs  如果客户端发现心跳超期，客户端会coordinator 为不可用，并阻塞心跳线程，如果超过poll 消息的间隔超过了 rebalanceTimeoutMs, 则 consumer 告知 broker 主动离开消费组，也会触发再平衡。 


# 再均衡

再均衡（ rebalance ）本质上是一种协议，规定了一个消费组所有消费者如何达成一致来分配订阅主题的每个分区。 

比如某个消费组有20 个消费组，订阅了一个具有100个分区的主题。正常情况下，kafka 平均会为每个消费分配5个分区。这个分配过程就是再均衡。

如何进行组内分区分配？ 

三种分配策略： RangeAssignor 和 RoundrobinAssignor 以及 StickyAssignor。

谁来执行再均衡和消费组管理

kafka 提供了一个角色： Group Coordinator 来执行对于消费组的管理。

Group Coordinator ：每个消费组分配一个消费组协调器用于组管理和位移管理。当消费组的第一个消费者启动的时候，它会去和kafka broker 确定谁是它们的组的组协调器。之后该消费组所有消费者和该组协调器协调通信。


# 幂等性

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

# 消费者位移管理



kafka 中，消费者根据消息的位移顺序消费消息。 

# 消费者管理

consumer group 是 kafka 提供可扩展且具有容错性的消费者机制。

三个特性：

1、消费组有一个或多个消费者，消费者可以是一个进程，也可以是一个线程。
2、group.id 是一个字符串，唯一标识一个消费组。
3、消费组订阅的主题每个分区只能分配给消费一个消费者。

消费者位移

消费者在消费的过程中记录已消费的数据，即消费位移（offset） 信息。
每个消费组只在自己的位移信息，那么只需要简单的一个整数表示位置就够了；同时可以引入 checkpoint 机制定期持久化。

# 位移管理 

kafka 默认定期自动提交位移，定期把 group 消费情况保存起来，做成一个 offset map 如下图所示：

![](/assets/mq/kafka-2023-03-13-22-31-41.png)


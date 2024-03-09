---
layout: post
title: Kafka 生产者和分区策略
date: 2022-02-27 10:12:15
categories: MQ  
tags: kafka 技术学习笔记
excerpt: Kafka 生产者的逻辑和分区策略
---

# Producer

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


# 分区策略

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

## 轮询策略

Round-robin 策略，就是按序分配，这是一种默认的策略。 如果 RroducerPecord 没有 partition 值又没有 key。

假如有四个 partition, 第一个消息分配到 分区 0， 第二消息分配到 分区1 依次类推。到第五个消息又分配到分区0。如下图： 

![](/assets/mq/kafka-mq-2023-02-28-18-05-23.png)

这种策略表现非常好，负载比较均衡。

## 随机策略

Randomness 策略。就是我们随意地将消息放置到任意一个分区上

![](/assets/mq/kafka-mq-2023-02-28-18-16-58.png)

实现随机策略方法: 拿到 topic 的 partition 数，然后用  ThreadLocalRandom 随机出一个比它小的数。

这种方法可能会出不均衡的情况。

```java
List partitions = cluster.partitionsForTopic(topic);
return ThreadLocalRandom.current().nextInt(partitions.size());
```
## 按消息键保序策略

Key-ordering 策略。就是为每一条消息定义一个消息键（key），可以有明确的业务含义，如果 业务编码、部门ID等。 

相同的消息键（key)会被放到相同的 partition 中。这个策略有一个缺点就是导致数据倾斜。 如下图：

![](/assets/mq/kafka-mq-2023-02-28-18-29-52.png)

代码实现是 用 key hash 之后 对 partition 取模。

```java
List partitions = cluster.partitionsForTopic(topic);
return Math.abs(key.hashCode()) % partitions.size();
```
在分区器上执行策略之后，就会被分发到对应的 partition 上。

# 缓冲区 

 [深度剖析 Kafka Producer 的缓冲池机制](https://cloud.tencent.com/developer/article/1698563)
这个篇文章写的很好，后面自己再分析一下。

--- 


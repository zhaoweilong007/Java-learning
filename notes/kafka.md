<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [Kafka](#kafka)
  - [相关概念](#%E7%9B%B8%E5%85%B3%E6%A6%82%E5%BF%B5)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# Kafka

Apache Kafka起源于LinkedIn，后来于2011年成为开源Apache项目，然后于2012年成为First-class Apache项目。Kafka是用Scala和Java编写的。

Apache Kafka 是一个分布式发布 - 订阅消息系统，可以处理大量的数据 Kafka 适合离线和在线消息消费。 Kafka 消息保留在磁盘上，并在群集内复制以防止数据丢失。 Kafka 构建在
ZooKeeper 同步服务之上。 它与 Apache Storm 和 Spark 非常好地集成，用于实时流式数据分析。 Kafka 是一个分布式消息队列，具有高性能、持久化、多副本备份、横向扩展能力

## 相关概念

- （1）生产者和消费者（producer和consumer）：消息的发送者叫 Producer，消息的使用者和接受者是 Consumer，生产者将数据保存到 Kafka
  集群中，消费者从中获取消息进行业务的处理。

- （2）broker：Kafka 集群中有很多台 Server，其中每一台 Server 都可以存储消息，将每一台 Server 称为一个 kafka 实例，也叫做 broker。

- （3）主题（topic）：一个 topic 里保存的是同一类消息，相当于对消息的分类，每个 producer 将消息发送到 kafka 中，都需要指明要存的 topic
  是哪个，也就是指明这个消息属于哪一类。

- （4）分区（partition）：每个 topic 都可以分成多个 partition，每个 partition 在存储层面是 append log 文件。任何发布到此 partition
  的消息都会被直接追加到 log 文件的尾部。分区可以使数据储存到不同的server上，通过partition实现了分布式储存

- （5）偏移量（Offset）：一个分区对应一个磁盘上的文件，而消息在文件中的位置就称为 offset（偏移量），offset 为一个 long 型数字，它可以唯一标记一条消息。由于kafka
  并没有提供其他额外的索引机制来存储 offset，文件只能顺序的读写，所以在kafka中几乎不允许对消息进行“随机读写”。

> 一个topic可以分成多个partition分散储存在多个broke上，每个partition对应一个文件，每个broke负责对本机上的partition进行读写
## 消息队列

#### 什么是消息队列？使用场景？

> RcoketMQ 是一款低延迟、高可靠、可伸缩、易于使用的消息中间件。具有以下特性：支持发布/订阅（Pub/Sub）和点对点（P2P）消息模型在一个队列中可靠的先进先出（FIFO）和严格的顺序传递支持拉（pull）和推（push）两种消息模式单一队列百万消息的堆积能力支持多种消息协议，如 JMS、MQTT 等分布式高可用的部署架构,满足至少一次消息传递语义提供 docker 镜像用于隔离测试和云集群部署提供配置、指标和监控等功能丰富的 Dashboard

- 相关概念

**NameServer**
消息服务中心，管理broker
**Producer**
消息的发送者
**Comsumer**
消息消费者
**Broker**
暂存和传输消息
**Topic**
区分消息的种类，一个发送者可以发送给给一个或多个topic，一个消费者可ui订阅一个或多个topic消息
**Message Queue**
相当于topic的分区，用于并行发送和接受消息

**工作流程**

- 启动NameServer，监听端口，等待Broker、producer、consumer连接
-

Broker启动，跟每个NameServer建立长连接，定时发送心跳包，心跳i包包含当前broker信息（IP+端口）以及存储所有topic信息，注册成功后，Nameserver集群中就有topic跟broker的映射关系了

- 收发消息前，先创建topic，创建topic需要指定该topic存储在那些broker上，也可以在发送消息时自动创建topic
- producer发送消息，启动时先跟Namserver集群中任意一台建立长连接，并从NameServer中获取当前发送的topic存在那些
  broker上，轮询从队列列表选择一个队列，然后与队列所在的broker建立长连接从而向broker发送消息
- consumer跟producer类似，跟其中一台NameServer建立长连接，获取当前topic存在那些broker上，然后直接跟broker建立长连接，开始消费数据

**消息样例**

- 基本样例
- 顺序消息
- 延时消息
- 批量消息
- 过滤消息
- 事务消息

**消息模式**

- 负载均衡
- 广播模式

消息储存

- commitlog
- consumerQueue
- indexFile

刷盘机制

- 同步刷盘
- 异步刷盘

消息同步机制

- 同步复制
- 异步复制

死信队列
> 将那些无法被消息者消息的消息放入一个队列中，这个列队叫死信队列

消费幂等性
> 对于消费者来说，不管是消费一次还是消费多次，最终的结果都是一样的

如何保证? 对于生产者而言，消息重发是不可避免的，所以需要在消息者处理，做幂等性校验，每次消费前，需要判断消息是否已经消费国过，比如通过记录消息的状态来判断

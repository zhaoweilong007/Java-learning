# RocketMQ入门教程

## 概述

了解rocketmq基础知识，请看我的 [rocketMQ学习](../notes/rocketMQ.md)
,或者移步到rocketmq的github地址 https://github.com/apache/rocketmq/tree/master/docs/cn

## 支持的特性

- 同步消息
- 异步消息
- 批量消息
- 顺序消费
- 延时消息
- 过滤消息
- 事务消息

## 安装部署

**环境说明**
rocketmq4.9.2 springboot2.5.6 jdk11 gradle7.3

使用docker部署nameServer、broke、rocketmq-console

参考[rocketmq-docker](https://github.com/apache/rocketmq-docker) 本地构建rocketmq镜像

构建镜像过程中会出错，需要修改下`Dockerfile-centos`脚本

将

```shell
RUN set -eux; \
    curl -L https://archive.apache.org/dist/rocketmq/${ROCKETMQ_VERSION}/rocketmq-all-${ROCKETMQ_VERSION}-bin-release.zip -o rocketmq.zip; \
    curl -L https://archive.apache.org/dist/rocketmq/${ROCKETMQ_VERSION}/rocketmq-all-${ROCKETMQ_VERSION}-bin-release.zip.asc -o rocketmq.zip.asc; \
    #https://www.apache.org/dist/rocketmq/KEYS
    curl -L https://www.apache.org/dist/rocketmq/KEYS -o KEYS; \
    \
    gpg --import KEYS; \
    gpg --batch --verify rocketmq.zip.asc rocketmq.zip ; \
    unzip rocketmq.zip ; \
    mv rocketmq-all*/* . ; \
    rmdir rocketmq-all*  ; \
    rm rocketmq.zip rocketmq.zip.asc KEYS
```

修改为

```shell
RUN set -eux; \
    curl -L https://archive.apache.org/dist/rocketmq/4.9.2/rocketmq-all-4.9.2-bin-release.zip -o rocketmq.zip; \
    curl -L https://archive.apache.org/dist/rocketmq/4.9.2/rocketmq-all-4.9.2-bin-release.zip.asc -o rocketmq.zip.asc; \
    #https://www.apache.org/dist/rocketmq/KEYS
        curl -L https://www.apache.org/dist/rocketmq/KEYS -o KEYS; \
        \
        gpg --import KEYS; \
    gpg --batch --verify rocketmq.zip.asc rocketmq.zip ; \
    unzip rocketmq.zip ; \
        mv rocketmq*/* . ; \
        rm -f rocketmq.zip rocketmq.zip.asc KEYS ;\
        rmdir rocketmq*  
```

再执行构建脚本

```shell
cd image-build && sh build-image.sh 4.9.2 centos
```

**docker-compose.yml**

```yaml
version: '3.5'

services:
  rmqnamesrv:
    image: apacherocketmq/rocketmq:4.9.2
    container_name: rmqnamesrv
    ports:
      - 9876:9876
    volumes:
      - ./rmqs/logs:/home/rocketmq/logs
      - ./rmqs/store:/home/rocketmq/store
    environment:
      JAVA_OPT_EXT: "-Duser.home=/home/rocketmq -Xms512M -Xmx512M -Xmn128m"
    command: [ "sh","mqnamesrv" ]
    networks:
      rmq:
        aliases:
          - rmqnamesrv
  rmqbroker:
    image: apacherocketmq/rocketmq:4.9.2
    container_name: rmqbroker
    ports:
      - 10909:10909
      - 10911:10911
    volumes:
      - ./rmq/logs:/home/rocketmq/logs
      - ./rmq/store:/home/rocketmq/store
      - ./rmq/brokerconf/broker.conf:/home/rocketmq/rocketmq-4.9.2/conf/broker.conf
    environment:
      JAVA_OPT_EXT: "-Duser.home=/home/rocketmq -server -Xms512M -Xmx512M -Xmn128m"
    command: [ "sh","mqbroker","-c","/home/rocketmq/rocketmq-4.9.2/conf/broker.conf","-n","rmqnamesrv:9876" ]
    depends_on:
      - rmqnamesrv
    networks:
      rmq:
        aliases:
          - rmqbroker

  rmqconsole:
    image: apacherocketmq/rocketmq-dashboard:1.0.0
    container_name: rmqconsole
    ports:
      - 8080:8080
    environment:
      JAVA_OPTS: "-Drocketmq.namesrv.addr=rmqnamesrv:9876 -Dcom.rocketmq.sendMessageWithVIPChannel=false"
    depends_on:
      - rmqnamesrv
    networks:
      rmq:
        aliases:
          - rmqconsole

networks:
  rmq:
    name: rmq
    driver: bridge

```

**broker.conf配置文件**

```properties
# Licensed to the Apache Software Foundation (ASF) under one or more
# contributor license agreements.  See the NOTICE file distributed with
# this work for additional information regarding copyright ownership.
# The ASF licenses this file to You under the Apache License, Version 2.0
# (the "License"); you may not use this file except in compliance with
# the License.  You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
#  Unless required by applicable law or agreed to in writing, software
#  distributed under the License is distributed on an "AS IS" BASIS,
#  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
#  See the License for the specific language governing permissions and
#  limitations under the License.
#所属集群名字
brokerClusterName=DefaultCluster
#broker名字，注意此处不同的配置文件填写的不一样，如果在broker-a.properties使用:broker-a,
#在broker-b.properties使用:broker-b
brokerName=broker-a
#0 表示Master，>0 表示Slave
brokerId=0
#nameServer地址，分号分割
#namesrvAddr=rocketmq-nameserver1:9876;rocketmq-nameserver2:9876
namesrvAddr=127.0.0.1:9876
#启动IP,如果 docker 报 com.alibaba.rocketmq.remoting.exception.RemotingConnectException: connect to <192.168.0.120:10909> failed
# 解决方式1 加上一句producer.setVipChannelEnabled(false);，解决方式2 brokerIP1 设置宿主机IP，不要使用docker 内部IP
brokerIP1=192.168.216.202
#在发送消息时，自动创建服务器不存在的topic，默认创建的队列数
defaultTopicQueueNums=4
#是否允许 Broker 自动创建Topic
autoCreateTopicEnable=true
#是否允许 Broker 自动创建订阅组，建议线下开启，线上关闭
autoCreateSubscriptionGroup=true
#Broker 对外服务的监听端口
listenPort=10911
#删除文件时间点，默认凌晨4点
deleteWhen=04
#文件保留时间，默认48小时
fileReservedTime=120
#commitLog每个文件的大小默认1G
mapedFileSizeCommitLog=1073741824
#ConsumeQueue每个文件默认存30W条，根据业务情况调整
mapedFileSizeConsumeQueue=300000
#destroyMapedFileIntervalForcibly=120000
#redeleteHangedFileInterval=120000
#检测物理文件磁盘空间
diskMaxUsedSpaceRatio=88
#存储路径
#storePathRootDir=/home/ztztdata/rocketmq-all-4.1.0-incubating/store
#commitLog 存储路径
#storePathCommitLog=/home/ztztdata/rocketmq-all-4.1.0-incubating/store/commitlog
#消费队列存储
#storePathConsumeQueue=/home/ztztdata/rocketmq-all-4.1.0-incubating/store/consumequeue
#消息索引存储路径
#storePathIndex=/home/ztztdata/rocketmq-all-4.1.0-incubating/store/index
#checkpoint 文件存储路径
#storeCheckpoint=/home/ztztdata/rocketmq-all-4.1.0-incubating/store/checkpoint
#abort 文件存储路径
#abortFile=/home/ztztdata/rocketmq-all-4.1.0-incubating/store/abort
#限制的消息大小
maxMessageSize=6291456
#flushCommitLogLeastPages=4
#flushConsumeQueueLeastPages=2
#flushCommitLogThoroughInterval=10000
#flushConsumeQueueThoroughInterval=60000
#Broker 的角色
#- ASYNC_MASTER 异步复制Master
#- SYNC_MASTER 同步双写Master
#- SLAVE
brokerRole=ASYNC_MASTER
#刷盘方式
#- ASYNC_FLUSH 异步刷盘
#- SYNC_FLUSH 同步刷盘
flushDiskType=ASYNC_FLUSH
#发消息线程池数量
sendMessageThreadPoolNums=128
#拉消息线程池数量
pullMessageThreadPoolNums=128

```

> 需要在docker-compose.yml当前目录创建rmq/logs、rmq/store、rmq/brokerconf、rmq/brokerconf/broker.conf目录及文件，broke配置文件种brokerIP1需要改成本地ip


使用docker-compose启动

```shell
docker-compose up -d
```

访问rocketmq的控制台<http://localhost:8080>,至此，mq部署完成

#### 项目搭建

新建rocketmq-consumer、rocketmq-provider两个项目

引入依赖

```groovy
    implementation 'org.apache.rocketmq:rocketmq-spring-boot-starter:2.2.1'
```

rocketmq-provider生产者配置

```yaml
server:
  port: 8112

spring:
  application:
    name: rocketmq-provider

rocketmq:
  producer:
    #消息组
    group: my-group
    #启动消息链路
    enable-msg-trace: true
    customized-trace-topic: my-trace-topic
    #配置ACL
    access-key: AK
    secret-key: SK
  name-server: 127.0.0.1:9876
```

rocketmq-consumer消费者配置

```yaml
server:
  port: 8111

spring:
  application:
    name: rocketmq-consumer

rocketmq:
  consumer:
    group: my-group
    enable-msg-trace: true
    customized-trace-topic: my-trace-topic
    access-key: AK
    secret-key: SK
  name-server: 127.0.0.1:9876
```

生产者编写一个command类在spring启动后运行

测试消息，我这里写的比较简单了

```java
package com.zwl.producer;

import cn.hutool.core.exceptions.ExceptionUtil;
import cn.hutool.core.lang.Snowflake;
import cn.hutool.core.util.RandomUtil;
import com.zwl.model.DemoMessage;
import java.util.List;
import java.util.Random;
import java.util.concurrent.atomic.AtomicInteger;
import java.util.function.Consumer;
import java.util.stream.Collectors;
import java.util.stream.Stream;
import lombok.extern.slf4j.Slf4j;
import org.apache.rocketmq.client.producer.SendCallback;
import org.apache.rocketmq.client.producer.SendResult;
import org.apache.rocketmq.spring.core.RocketMQTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.CommandLineRunner;
import org.springframework.messaging.support.MessageBuilder;
import org.springframework.stereotype.Component;

/**
 * @author ZhaoWeiLong
 * @since 2021/11/25
 */
@Component
@Slf4j
public class producerRunner implements CommandLineRunner {

  @Autowired
  RocketMQTemplate rocketMQTemplate;

  Snowflake snowflake = new Snowflake(1, 1, true);

  @Override
  public void run(String... args) throws Exception {
    final List<DemoMessage> list =
        Stream.generate(
                () -> {
                  final DemoMessage message = new DemoMessage();
                  message.setId(snowflake.nextId());
                  return message;
                })
            .limit(RandomUtil.randomInt(10, 100))
            .collect(Collectors.toList());

    sendSyncMsg.accept(list);

    sendAsyncMsg.accept(list);

    sendOrderlyMsg.accept(list);

    sendBatchMsg.accept(list);

    sendTransactionMsg.accept(list);

    log.info("=============消息全部发送完成===================");
  }

  /** 发送同步消息 */
  Consumer<List<DemoMessage>> sendSyncMsg =
      new Consumer<List<DemoMessage>>() {
        @Override
        public void accept(List<DemoMessage> list) {
          list.forEach(
              demoMessage -> {
                demoMessage.setDesc("同步消息");
                rocketMQTemplate.convertAndSend("test-topic-1", demoMessage);
              });
        }
      };

  /** 发送异步消息 */
  Consumer<List<DemoMessage>> sendAsyncMsg =
      new Consumer<List<DemoMessage>>() {
        @Override
        public void accept(List<DemoMessage> list) {
          list.forEach(
              demoMessage -> {
                demoMessage.setDesc("异步消息");
                rocketMQTemplate.asyncSend(
                    "test-topic-2",
                    MessageBuilder.withPayload(demoMessage),
                    new SendCallback() {
                      @Override
                      public void onSuccess(SendResult sendResult) {
                        log.info("异步消息发送成功:{}", sendResult);
                      }

                      @Override
                      public void onException(Throwable e) {
                        log.info("异步消息发送失败:{}", ExceptionUtil.stacktraceToString(e));
                      }
                    });
              });
        }
      };

  final Random random = new Random();

  /** 发送顺序消息 */
  Consumer<List<DemoMessage>> sendOrderlyMsg =
      new Consumer<List<DemoMessage>>() {
        AtomicInteger count = new AtomicInteger(0);

        @Override
        public void accept(List<DemoMessage> list) {
          list.forEach(
              demoMessage -> {
                demoMessage.setDesc("顺序消息" + count.incrementAndGet());
                rocketMQTemplate.syncSendOrderly(
                    "test-topic-3", MessageBuilder.withPayload(demoMessage), "hashKey");
              });
        }
      };
  /** 发送批量消息 */
  Consumer<List<DemoMessage>> sendBatchMsg =
      new Consumer<List<DemoMessage>>() {
        @Override
        public void accept(List<DemoMessage> list) {
          list.forEach(
              demoMessage -> {
                demoMessage.setDesc("批量消息" + System.currentTimeMillis());
              });
          rocketMQTemplate.convertAndSend("test-topic-4", list);
        }
      };

  Consumer<List<DemoMessage>> sendTransactionMsg =
      new Consumer<List<DemoMessage>>() {
        @Override
        public void accept(List<DemoMessage> list) {
          list.forEach(
              demoMessage -> {
                demoMessage.setDesc("事务消息" + System.currentTimeMillis());
                rocketMQTemplate.sendMessageInTransaction(
                    "test-topic-5", MessageBuilder.withPayload(demoMessage).build(), null);
              });
        }
      };
}

```

mq消费者

```java
package com.zwl.consumer;

import com.zwl.model.DemoMessage;
import java.util.List;
import lombok.extern.slf4j.Slf4j;
import org.apache.rocketmq.spring.annotation.ConsumeMode;
import org.apache.rocketmq.spring.annotation.RocketMQMessageListener;
import org.apache.rocketmq.spring.annotation.RocketMQTransactionListener;
import org.apache.rocketmq.spring.core.RocketMQListener;
import org.apache.rocketmq.spring.core.RocketMQLocalTransactionListener;
import org.apache.rocketmq.spring.core.RocketMQLocalTransactionState;
import org.springframework.messaging.Message;
import org.springframework.stereotype.Component;

/**
 * @author ZhaoWeiLong
 * @since 2021/11/25
 */
@Slf4j
public class ConsumerReceiver {

  @RocketMQMessageListener(topic = "test-topic-1", consumerGroup = "${rocketmq.consumer.group}")
  @Component
  public static class SyncConsumer implements RocketMQListener<DemoMessage> {

    @Override
    public void onMessage(DemoMessage message) {
      log.info("线程：{}，SyncConsumer-msg:{}", Thread.currentThread().getName(), message);
    }
  }

  @RocketMQMessageListener(topic = "test-topic-2", consumerGroup = "${rocketmq.consumer.group}")
  @Component
  public static class AsyncConsumer implements RocketMQListener<DemoMessage> {

    @Override
    public void onMessage(DemoMessage message) {
      log.info("线程：{}，AsyncConsumer-msg:{}", Thread.currentThread().getName(), message);
    }
  }

  @RocketMQMessageListener(
      topic = "test-topic-3",
      consumerGroup = "${rocketmq.consumer.group}",
      consumeMode = ConsumeMode.ORDERLY)
  @Component
  public static class OrderlyConsumer implements RocketMQListener<DemoMessage> {

    @Override
    public void onMessage(DemoMessage message) {
      log.info("线程：{}，OrderlyConsumer-msg:{}", Thread.currentThread().getName(), message);
    }
  }

  @RocketMQMessageListener(topic = "test-topic-4", consumerGroup = "${rocketmq.consumer.group}")
  @Component
  public static class BatchConsumer implements RocketMQListener<List<DemoMessage>> {

    @Override
    public void onMessage(List<DemoMessage> messages) {
      log.info("线程：{}，BatchConsumer-msg:{}", Thread.currentThread().getName(), messages);
    }
  }

  @RocketMQTransactionListener
  @Component
  public static class TransactionConsumer implements RocketMQLocalTransactionListener {

    /**
     * 执行本地事务
     *
     * @param msg
     * @param arg
     * @return
     */
    @Override
    public RocketMQLocalTransactionState executeLocalTransaction(Message msg, Object arg) {
      log.info("invoke executeLocalTransaction msg:{}", msg);
      return RocketMQLocalTransactionState.UNKNOWN;
    }

    /**
     * 检车事务状态
     *
     * @param msg
     * @return
     */
    @Override
    public RocketMQLocalTransactionState checkLocalTransaction(Message msg) {
      log.info("invoke checkLocalTransaction msg:{}", msg);
      return RocketMQLocalTransactionState.COMMIT;
    }
  }
}

```
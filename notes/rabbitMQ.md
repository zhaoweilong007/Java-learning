# RabbitMq

> 对应项目 [springboot-matrix](https://github.com/zhaoweilong007/spring-boot-matrix) 下`rabbitmq-consumer`、`rabbitmq-provider`

## 概念

> 由erlang语言开发，基于AMQP协议实现的队列，消息队列负责储存和转发消息，消息队列主要用于异步、削峰、解耦

### channel

> 与mq建立连接的通道，是建立在TCP上的虚拟通道，多个通道可以在一条TCP上处理

### exchange

> 交换机，负责根据routeKey路由到不同的队列

**四种交换机种类**

- direct 直连交换机，rabbitmq的默认交换机，绑定此交换机需指定routeKey，交换机根据routeKey路由到指定队列
- fanout 扇形交换机 忽略routeKey，讲消息广播到所有与其绑定的队列
- header 首部交换机，忽略routeKey，根据Headers信息进行匹配，header是一个map，可以匹配其他类型数据，如所有键值对匹配或单一键值对匹配
- topic 主题交换机，在direct模式上增加模式匹配，对routeKey进行模式匹配，*代表一个单词，#代表一个或多个

## 工作模式

### 简单模式(simple)

一个生产者对应一个消费者，一对一

### 工作队列(work queue)

多个消费者之间竞争消费，单生产者，多消费者，单队列

### 发布订阅（publish/subscribe）

消息广播，单生产者，多消费者，多队列

### 路由模式（routeing）

根据routeingKey有选择性的接收消息，每个队列通过routeing Key需要全文匹配

### 主题模式（topics）

根据routeing Key和模式进行匹配，多消费者，多队列，每个队列通过模式匹配

### RPC模式

发布者发送消息，需要等待消费者消费完返回结果，类似RPC远程过程调用，只不过是通过mq完成而不是直接建立连接

## 配置

application.yml

```yaml
spring:
  application:
    name: rabbitmq-provider
  rabbitmq:
    host: 127.0.0.1
    port: 5672
    username: guest
    password: guest
    cache:
      # 当缓存模式设置为CacheMode.CONNECTION时，连接会被缓存起来，createConnection()的调用可能会创建一个新的连接，
      #也可能从缓存中获取；关闭连接时，如果缓存的连接数没有达到cache size，就会将连接缓存起来。
      #在这个模式下，由连接创建的channel也会被缓存起来。
      # CacheMode.CHANNEL模式下，所有的客户端共享一个链接，不同的channel之间相互隔离
      connection:
        mode: channel
      channel:
        size: 2048
        checkout-timeout: 3000
    connection-timeout: 3000
server:
  port: 9871
```

## 实践

### 部署

下载rabbitmq，从[官网]()获取下载方式

我在windows系统使用scoop下载rabbitmq

安装rabbitmq

```shell
scoop install rabbitmq
```

启动服务

```shell
rabbitmq-server start -detached
```

启动rabbitmq管理插件

```shell
rabbitmq-plugins enable rabbitmq_management
```

访问 <http://127.0.0.1:5672>进入rabbitmq的管理界面，默认用户名密码`guest`



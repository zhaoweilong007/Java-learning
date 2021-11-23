# 分布式事务Seata

> 对应项目 [springboot-matrix](https://github.com/zhaoweilong007/spring-boot-matrix) 下`seata-dubbo-interface`、`seata-order-service`、`seata-storage-service`、`seata-account-service`

## 分布式事务基本概念

> 在分布式系统中，一次操作由多个系统协同完成，这种一次操作协同多个系统完成的过程叫做分布式事务

### CAP理论

> 在一个分布式系统中不可能同时满足一致性、可用性、分区容错性，最多满足两个，对于分布式系统而言，必须保证P，要么CP，要么AP

- 一致性 指强一致性，数据始终都是一致的
- 可用性 系统提供的服务一直处于可用状态，用户的操作请求在指定时间内响应请求，超出时间范围，认为系统不可用
- 分区容错性 分布式系统在遇到任何网络分区故障的时候，仍需要保证对外提供一致性和可用性，除非整个网络发生故障

**网络分区**
在分布系统中，可能由于故障导致节点之间无法连通，整个网络分成了几块区域

zookeeper采用的CP架构，eureka是AP架构，Nacos支持CP、AP架构

### BASE理论

> 在cap理论上权衡的一种结果，即使做不到强一致性，但是可以做到最终一致性

- 基本可用
- 软状态
- 最终一致性

### 2PC（二阶段提交）

- 阶段一:提交事务请求（prepare）

1、协调者向所有参与者发送事务内容，询问是否可以提交，并等待参与者反馈

2、参与者执行本地事务，讲执行结果反馈给协调者

- 阶段二:执行事务提交（commit）

1、根据各参与者的反馈结果，是否执行事务提交，否则中断事务提交，回滚事务

**2PC问题**

- 同步阻塞

  协调者要等待所有参与者反馈信息后才能继续执行
- 单点问题

  协调者挂掉导致的事务一致性的问题
- 脑裂问题

  网络分区问题，网络不好导致其中一个参与者挂掉事务不一致的问题

### 3PC（三阶段提交）

> 为了解决2PC的问题，引用了超时机制

- 阶段一 canCommit（准备阶段）

  1、事务询问，是否可以执行事务

  2、各参与者反馈事务询问的响应

- 阶段二 preCommit（预提交）

  根据阶段一的反馈结果分为两种情况

  1、 执行事务预提交

  协调者向所有参与者发送preCommit请求，各参与者接收到预提交，执行事务操作，将执行结果反馈给协调者

  2、中断事务

  协调者向所有参与者发送abort请求，各参与者收到请求或者超时，都会中断事务


- 阶段三 doCommit（提交）

  1、执行提交 协调者发送doCommit请求，各参与者收到请求执行提交事务，反馈ack给协调者，协调者收到各参与者ack，事务完成 2、中断事务
  协调者向所有参与者发送abort请求，参与者收到请求，根据二阶段的undo信息进行事务回滚操作，回滚完成反馈ack消息，协调者收到 所有参与者ack消息，中断事务

3pc相较于2pc解决了阻塞和单点问题，但是还是无法解决脑裂问题

### TCC（补偿性事务）

> TCC补偿性事务是基于2PC实现的业务层事务控制方案，可以保证数据最终一致性

主要有三个方法，try、confirm、cancel

- 1、try检查及预留业务资源完成提交事务前检查，并预留资源
- 2、确认事务提交
- 3、取消事务提交，回滚事务

**缺点**

- 对代码侵入性强
- 需要在try、confirm接口实现幂等性（一个操作无论执行多少次，结果都是一样的）

### 消息事务

> 基于消息队列+消息表实现的分布式事务，如rocketmq执行的分布式事务，保证数据的最终一致性

1、发送方给broker发送半消息，半消息对订阅方不可见，发送成功后执行本地事务，根据事务结果执行commit或者rollback，
发送方需要提交一个反查事务接口，用于broker检查事务是否执行成功

2、如果发送发commit成功，订阅方就可以收到这条消息，执行本地事务，消费消息

### Seata事务

> 开源的分布式事务一致性解决方案，seata提供了AT、TCC、XA、SAGA事务模式

#### 三种角色

- TC (Transaction Coordinator) - 事务协调者：维护全局和分支事务的状态，驱动全局事务提交或回滚。
- TM (Transaction Manager) - 事务管理器：定义全局事务的范围，开始全局事务、提交或回滚全局事务。
- RM ( Resource Manager ) - 资源管理器：管理分支事务处理的资源( Resource )，与 TC 交谈以注册分支事务和报告分支事务的状态，并驱动分支事务提交或回滚。

#### AT模式

> AT 模式是一种无侵入的分布式事务解决方案。在 AT 模式下，用户只需关注自己的“业务 SQL”，用户的 “业务 SQL” 作为一阶段，Seata 框架会自动生成事务的二阶段提交和回滚操作。

**前提**

- 基于支持本地 ACID 事务的关系型数据库。
- Java 应用，通过 JDBC 访问数据库。

**一阶段**

seata会拦截sql，解析元数据，在执行sql前保存before image、然后执行sql,执行sql后保存after
image，讲前后镜像数据以及业务sql组成一条回滚日志记录，插入到undo表中， 生成行锁，以上全部在一个事务中完成，保证一阶段的原子性

**二阶段提交**

如是二阶段是提交的话，因为sql已经在一阶段提交到数据库了，所以只需要删除一阶段保存的快照和undo log即可，请求会放在异步队列中执行

**二阶段回滚**

如果二阶段是回滚，则根据一阶段保存的快照before image还原业务数据，在还原之前，需要根据对比当前数据和after image中的数据是否一致，
如果一致则没有脏写，可以还原，或者根据配置策略处理

**写隔离**

- 阶段一在事务提交前，需要拿到全局锁
- 拿不到全局锁不能提交本地事务
- 拿全局锁的尝试被限制在一定范围内，超出范围将放弃，并回滚本地事务，释放本地锁。

**读隔离**

- seata默认隔离级别是读未提交
- 如需要求读已提交，使用select FROM UPDATE语句

## seata实践

> 使用版本，springboot2.5.6,seata-spring-boot-starter 1.4.2，dubbo-spring-boot-starter3.0.4，mysql:mysql-connector-java 8.0.22

### 准备

> nacos服务器版本：2.0.2， seata服务器版本：1.4.2

- 下载nacos，以单机模式部署nacos

- 下载seata，更新conf目录下的file.conf和registry.conf配置

我的配置如下

**file.cnf**

```json
## transaction log store, only used in seata-server
store {
## store mode: file、db、redis
mode = "db"
## rsa decryption public key
publicKey = ""
## file store property
file {
## store location dir
dir = "sessionStore"
# branch session size, if exceeded first try compress lockkey, still exceeded throws exceptions
maxBranchSessionSize = 16384
# globe session size, if exceeded throws exceptions
maxGlobalSessionSize = 512
# file buffer size, if exceeded allocate new buffer
fileWriteBufferCacheSize = 16384
# when recover batch read size
sessionReloadReadSize = 100
# async, sync
flushDiskMode = async
}

## database store property
db {
## the implement of javax.sql.DataSource, such as DruidDataSource(druid)/BasicDataSource(dbcp)/HikariDataSource(hikari) etc.
datasource = "druid"
## mysql/oracle/postgresql/h2/oceanbase etc.
dbType = "mysql"
driverClassName = "com.mysql.cj.jdbc.Driver"
## if using mysql to store the data, recommend add rewriteBatchedStatements=true in jdbc connection param
url = "jdbc:mysql://127.0.0.1:3306/seata?rewriteBatchedStatements=true"
user = "root"
password = "123456"
minConn = 5
maxConn = 100
globalTable = "global_table"
branchTable = "branch_table"
lockTable = "lock_table"
queryLimit = 100
maxWait = 5000
}

## redis store property
redis {
## redis mode: single、sentinel
mode = "single"
## single mode property
single {
host = "127.0.0.1"
port = "6379"
}
## sentinel mode property
sentinel {
masterName = ""
## such as "10.28.235.65:26379,10.28.235.65:26380,10.28.235.65:26381"
sentinelHosts = ""
}
password = ""
database = "0"
minConn = 1
maxConn = 10
maxTotal = 100
queryLimit = 100
}
}
```

**registry.conf**

```json
registry {
  # file 、nacos 、eureka、redis、zk、consul、etcd3、sofa
type = "nacos"

nacos {
application = "seata-server"
serverAddr = "127.0.0.1:8848"
group = "SEATA_GROUP"
namespace = "seata"
cluster = "default"
username = "nacos"
password = "nacos"
}
eureka {
serviceUrl = "http://localhost:8761/eureka"
application = "default"
weight = "1"
}
redis {
serverAddr = "localhost:6379"
db = 0
password = ""
cluster = "default"
timeout = 0
}
zk {
cluster = "default"
serverAddr = "127.0.0.1:2181"
sessionTimeout = 6000
connectTimeout = 2000
username = ""
password = ""
}
consul {
cluster = "default"
serverAddr = "127.0.0.1:8500"
aclToken = ""
}
etcd3 {
cluster = "default"
serverAddr = "http://localhost:2379"
}
sofa {
serverAddr = "127.0.0.1:9603"
application = "default"
region = "DEFAULT_ZONE"
datacenter = "DefaultDataCenter"
cluster = "default"
group = "SEATA_GROUP"
addressWaitTime = "3000"
}
file {
name = "file.conf"
}
}

config {
# file、nacos 、apollo、zk、consul、etcd3
type = "nacos"

nacos {
serverAddr = "127.0.0.1:8848"
namespace = "seata"
group = "SEATA_GROUP"
username = "nacos"
password = "nacos"
dataId = "seataServer.properties"
}
consul {
serverAddr = "127.0.0.1:8500"
aclToken = ""
}
apollo {
appId = "seata-server"
## apolloConfigService will cover apolloMeta
apolloMeta = "http://192.168.1.204:8801"
apolloConfigService = "http://192.168.1.204:8080"
namespace = "application"
apolloAccesskeySecret = ""
cluster = "seata"
}
zk {
serverAddr = "127.0.0.1:2181"
sessionTimeout = 6000
connectTimeout = 2000
username = ""
password = ""
nodePath = "/seata/seata.properties"
}
etcd3 {
serverAddr = "http://localhost:2379"
}
file {
name = "file.conf"
}
}


```

### 导入数据脚本

创建`seata`数据库，执行sql脚本

```sql
-- -------------------------------- The script used when storeMode is 'db' --------------------------------
-- the table to store GlobalSession data
CREATE TABLE IF NOT EXISTS `global_table`
(
    `xid`                       VARCHAR(128) NOT NULL,
    `transaction_id`            BIGINT,
    `status`                    TINYINT      NOT NULL,
    `application_id`            VARCHAR(32),
    `transaction_service_group` VARCHAR(32),
    `transaction_name`          VARCHAR(128),
    `timeout`                   INT,
    `begin_time`                BIGINT,
    `application_data`          VARCHAR(2000),
    `gmt_create`                DATETIME,
    `gmt_modified`              DATETIME,
    PRIMARY KEY (`xid`),
    KEY `idx_gmt_modified_status` (`gmt_modified`, `status`),
    KEY `idx_transaction_id` (`transaction_id`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8;

-- the table to store BranchSession data
CREATE TABLE IF NOT EXISTS `branch_table`
(
    `branch_id`         BIGINT       NOT NULL,
    `xid`               VARCHAR(128) NOT NULL,
    `transaction_id`    BIGINT,
    `resource_group_id` VARCHAR(32),
    `resource_id`       VARCHAR(256),
    `branch_type`       VARCHAR(8),
    `status`            TINYINT,
    `client_id`         VARCHAR(64),
    `application_data`  VARCHAR(2000),
    `gmt_create`        DATETIME(6),
    `gmt_modified`      DATETIME(6),
    PRIMARY KEY (`branch_id`),
    KEY `idx_xid` (`xid`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8;

-- the table to store lock data
CREATE TABLE IF NOT EXISTS `lock_table`
(
    `row_key`        VARCHAR(128) NOT NULL,
    `xid`            VARCHAR(128),
    `transaction_id` BIGINT,
    `branch_id`      BIGINT       NOT NULL,
    `resource_id`    VARCHAR(256),
    `table_name`     VARCHAR(32),
    `pk`             VARCHAR(36),
    `gmt_create`     DATETIME,
    `gmt_modified`   DATETIME,
    PRIMARY KEY (`row_key`),
    KEY `idx_branch_id` (`branch_id`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8;

CREATE TABLE IF NOT EXISTS `distributed_lock`
(
    `lock_key`   CHAR(20)    NOT NULL,
    `lock_value` VARCHAR(20) NOT NULL,
    `expire`     BIGINT,
    primary key (`lock_key`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8mb4;

INSERT INTO `distributed_lock` (lock_key, lock_value, expire)
VALUES ('AsyncCommitting', ' ', 0);
INSERT INTO `distributed_lock` (lock_key, lock_value, expire)
VALUES ('RetryCommitting', ' ', 0);
INSERT INTO `distributed_lock` (lock_key, lock_value, expire)
VALUES ('RetryRollbacking', ' ', 0);
INSERT INTO `distributed_lock` (lock_key, lock_value, expire)
VALUES ('TxTimeoutCheck', ' ', 0);
```

在各业务系统database上创建`undo_log`表

```mysql
CREATE TABLE undo_log
(
    id            BIGINT(20)   NOT NULL AUTO_INCREMENT,
    branch_id     BIGINT(20)   NOT NULL,
    xid           VARCHAR(100) NOT NULL,
    context       VARCHAR(128) NOT NULL,
    rollback_info LONGBLOB     NOT NULL,
    log_status    INT(11)      NOT NULL,
    log_created   DATETIME     NOT NULL,
    log_modified  DATETIME     NOT NULL,
    PRIMARY KEY (id),
    UNIQUE KEY ux_undo_log (xid, branch_id)
) ENGINE = InnoDB
  AUTO_INCREMENT = 1
  DEFAULT CHARSET = utf8;
```

- 启动nacos、seata服务
- 创建三个服务，order-service、storage-service、account-service服务模块
- gradle依赖如下

```groovy
dependencies {
    implementation(project(":seata-dubbo-interface"))
    implementation("com.baomidou:mybatis-plus-boot-starter")
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation("mysql:mysql-connector-java")
    implementation 'org.apache.dubbo:dubbo-spring-boot-starter'
    implementation 'org.apache.dubbo:dubbo-registry-nacos'
    implementation 'io.seata:seata-spring-boot-starter'
    testImplementation("org.springframework.boot:spring-boot-starter-test")
}
```

application.yml配置文件如下,这里只展示order服务的配置

```yaml
spring:
  application:
    name: order-service
  datasource:
    url: jdbc:mysql://localhost:3306/seata_order?allowPublicKeyRetrieval=true&useSSL=false&useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&useSSL=false&zeroDateTimeBehavior=convertToNull&serverTimezone=Asia/Shanghai
    driver-class-name: com.mysql.cj.jdbc.Driver
    username: root
    password: 123456
server:
  port: 8090

mybatis-plus:
  mapper-locations: classpath*:/mapper/*.xml
  typeAliasesPackage: com.zwl.domain
  configuration:
    map-underscore-to-camel-case: true
  global-config:
    db-config:
      id-type: auto

#dubbo配置
dubbo:
  application:
    id: ${spring.application.name}
    name: ${spring.application.name}
    qos-enable: false
    version: 1.0.0
  registry:
    id: ${spring.application.name}-registry
    address: nacos://127.0.0.1:8848
    username: nacos
    password: nacos
    check: true
    simplified: true
  config-center:
    address: nacos://127.0.0.1:8848
    check: true
    username: nacos
    password: nacos
  metadata-report:
    address: nacos://127.0.0.1:8848
    username: nacos
    password: nacos
  protocol:
    id: dubbo
    name: dubbo
    port: -1
  provider:
    timeout: 1000
    version: 1.0.0
  consumer:
    timeout: 1000
    version: 1.0.0
  scan:
    base-packages: com.zwl

#seata配置
seata:
  enabled: true
  data-source-proxy-mode: AT
  enable-auto-data-source-proxy: true
  application-id: ${spring.application.name}
  tx-service-group: ${spring.application.name}-group
  service:
    vgroup-mapping:
      order-service-group: default
    grouplist:
      default: 127.0.0.1:8091
  registry:
    type: nacos
    nacos:
      cluster: default
      application: seata-server
      server-addr: 127.0.0.1:8848
      namespace: seata
      group: SEATA_GROUP
      username: nacos
      password: nacos
```


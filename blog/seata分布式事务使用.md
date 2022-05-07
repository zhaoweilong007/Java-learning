<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [seata](#seata)
  - [seata实践](#seata%E5%AE%9E%E8%B7%B5)
    - [准备](#%E5%87%86%E5%A4%87)
    - [导入数据脚本](#%E5%AF%BC%E5%85%A5%E6%95%B0%E6%8D%AE%E8%84%9A%E6%9C%AC)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# seata

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


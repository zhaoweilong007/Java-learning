# Redis笔记

> redis是key-value的内存数据库，常用作缓存、分布式锁、消息队列，支持事务、持久化、lua脚本、多种集群方案

## redis数据类型和数据结构

- **string**

> string类型是基于动态数据SDS实现的

`set key value` 设置key-value
`mset key1 value1 key2 value2` 设置多个key-value
`get key` 获取对应value
`exists key` 是否存在
`strlen key` value字符串长度
`del key` 删除key
`mget key1 key2` 获取多个key
`incr key` 自增
`decr key` 自减
`expire key time` 设置过期时间
`setex key time value` 设置key并设置过期时间
`ttl key` 查看过期时间

- **list**

> list使用双向链表的数据结构

`lpush/lpop` 从队列左边添加元素/从队列左边弹出元素

`rpush/rpop` 从队列右边添加元素/从队列右边弹出元素

`lrange list 0 1` 查看对应下标的元素

`llen list` 查看list长度

`rpush/lpop` 实现队列

`rpusqh/rpop` 实现栈

- **hash**

> 类似于jdk的数据+链表，hash是一个String类型的field和value的映射表

`hset key key1 value1`

`hget key key1`

`hmset key key1 val1 key2 val2`

`hgettall key` 获取所有字段和值

`hkeys key` 获取所有key列表

`hvals key` 获取所有value列表

- **set**

> set是一种无序集合

`sadd myset val1 val2`
`smembers myset` 查看所有元素
`scard myset` 查看set长度
`sismember myset val1` 检查元素是否存在于set中
`sinterstore myset3 myset1 myset1` 获取myset1和myset2的交集存放在myset3中

- **zset**

> 有序集合，使用场景，排行耪、弹幕消息

`zadd myset score val1` 添加权重为score的val1到myset中

`zcard myset` 查看元素数量

`zscore myset val1` 查看元素的权重

`zrange myset 0 -1` 查看范围内的元素

`zrevrange myset 0 1` 逆序输出元素

- **bitmap**

> 存储连续的二进制数字(0、1),可以用来标识某写元素的状态，用户签到、用户登录

`setbit key 1 1`
`getbit key 1`
`bitcount key` 统计设置为1的数量

## rediso常见问题

### 缓存穿透

一直访问一个不存在的key，导致每次都要查数据库，造成数据库的压力增加

解决方案：不存在的key是作为缓存，或者使用布隆过滤器

### 缓存雪崩

大量的key同时过期，导致请求全部落在数据库

解决方案：对key分为冷、热key，在过期时间上增加随机时间

### redis双写一致性

Cache Aside Pattern 模式 简单来说，就是先更新数据库再删除缓存

- 为什么是删除缓存而不是更新

可能这个key不是频繁访问，不需要每次都更新，而删除后，在获取的时候再加载，有点懒加载的思想

### redis内存淘汰策略

- volatile-lru 已设置过期时间最少使用

- volatile-lfu 已设置过期时间最不经常使用

- volatile-ttl 已设置过期时间快要过期

- volatile-random 已设置过期时间中随机

- allkeys-lru 所有key中最少使用

- allkeys-lfu 移除最不经常使用

- allkeys-random 所有key中随机

- no-evication 禁止驱逐策略，默认策略

### redis持久化机制

- RDB 将某个时间点的所有数据都存放在硬盘上，可以将快照复制到其他服务器 缺点是:
  1、 如果服务器故障，会丢失最后一次创建快照之后的数据 2、如果数据量很大，保存的快照的时间会很长
- AOF 将写命令添加到AOF文件的末尾，将写命令存储到缓存区，再根据一定时间将缓存区写入到磁盘 有三个同步选项 appendfsync always
  #每次有数据修改发生时都会写入AOF文件,这样会严重降低Redis的速度 appendfsync everysec #每秒钟同步一次，显示地将多个写命令同步到硬盘 appendfsync no
  #让操作系统决定何时进行同步

### redis事务

> redis事务提供了一种将多个命令打包的功能，然后再按顺序执行，并且不会被中途打断，事务不支持原子性和回滚

- multi
- exec
- discard
- watch

## redis部署

### 主从模式

### 哨兵模式
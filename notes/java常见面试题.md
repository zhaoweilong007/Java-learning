# Java常见面试题

## GC分哪两种？minor Gc和full Gc有什么区别？什么时候触发full Gc？分别采用什么算法？

minor Gc在新生代
full Gc在老年代


- 标记清除
- 复制
- 标记整理
- 分代收集

## JVM有几种classload？为什么有这么多？
- 启动类加载器
- 扩展类加载类
- 系统类加载器
- 自定义类加载器

## 什么是双亲委派模型？有什么好处？什么情况下需要破坏双亲委派模型？
到类加载器收到一个类加载请求时，他并不是自己加载，而是交由他的父类加载器加载，依次递归，直到顶层类加载加载，顶层类能加载则直接返回，否则依次由其子类加载，
好处：避免类的重复加载，防止Java核心API被修改

## 常见的JVM调优？可以具体到哪个参数？


## HashMap数据结构？底层怎么实现的？并发问题？concurrentHashMap?怎么实现的？

HashMap主要用来存储键值对，基于哈希表的Map接口实现

jdk1.8之前，采用数据+链表的方式实现，1.8之后采用数组+链表/红黑树实现
默认容量是16，负载因子0.75，当链表长度大于8，数据长度大于64，链表会自动转换为红黑树，长度小于6，红黑树自动转换为链表，当数据长度大于16*0.75=12时，会自动进行扩容，每次扩容为原来的2倍

## 了解LinedHashMap的应用吗？
是有序的HashMap的实现，

## 什么时候出现线程僵死？


## 如何合理配置线程池大小？

## ThreadLocal、volatile的实现原理和使用场景？
ThreadLocal是一个线程副本变量，存储了每个线程的变量副本，线程与线程之间独立互不影响
ThreadLocal有一个静态内部类，ThreadLocalMap，key是线程的引用

volatile是jvm的一个关键字，volatile主要有两个作用，一是保证了原子性，二是禁止指令重排

## ThreadLocal什么情况会出现OOM？
当线程一直没有销毁，而使用完没有手动清除ThreadLocal的强引用时，key和value不断增加，最终导致OOM


## Spring有哪些优势？
重要有两个特性，Ioc和AOP

## Springboot相对于spring有哪些改进？spring5比spring4做了那些改进？
springboot实现了spring的自动配置，约定大于配置，相当与spring的一个脚手架

spring5特性：


## SpringBoot的自动装配原理？
注解SpringBootApplication实际上是一个复合注解
主要有两个主要的注解：
@AutoConfiguration和@AutoCompontScan

@AuotConfiguration注解会自动扫描MATE-INF下spring.factories文件，每个文件有个AutoConfig的配置类，通过加载这些自动类从而实现了spring的自动装配，@AutoCompontScan注解自动扫描包下面的组件注入到spring

## SpringApplication注解原理？

以上
## 消息中间件产品的优缺点？
主流消息中间件：rabbitMQ，RocketMQ，

## 消息中间件如何保证一致性？ 如何进行消息的重试的机制？
- 强一致性
- 最终一致性

## Redis为什么这么快？Redis采用多线程会有那些问题？
redis时基于内存的操作，存储结构简单，key-value形式
多线程可能会有并发问题

## Redis的锁的原子性操作？Redis内部是如何实现的？


## Spring Cloud和Dubbo下？什么场景下适合用Spring Cloud的？


## 锁机制介绍：行锁、表锁、共享锁、排他锁
- 行锁
对某一行数据进行
- 表锁
对整个表进行枷锁
- 共享锁
读读并行
- 排他锁
与其他锁互斥
- 间隙锁
- 自增锁

## 乐观锁的业务场景以及实现方式？
版本号机制
CAS无锁算法，及比较并交换，有三个值，内存值V，旧的预期值A，新的预期值B，当且仅当内存值等于旧的预期值，将内存值替换为新的预期值B
## 分布式事务的理解？二阶段提交？三阶段提交？
CAP理论
- 在一个分布式系统中，不可能同时满足一致性、可用性、分区容错性
BASE理论
基本可用、软状态、最终一致性

**二阶段提交**
- 一阶段



- 二阶段

**三阶段提交**
- 一阶段提交
- 二阶段提交
- 三阶段提交
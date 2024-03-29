<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [多线程](#%E5%A4%9A%E7%BA%BF%E7%A8%8B)
  - [ThreadLocal、volatile的实现原理和使用场景？](#threadlocalvolatile%E7%9A%84%E5%AE%9E%E7%8E%B0%E5%8E%9F%E7%90%86%E5%92%8C%E4%BD%BF%E7%94%A8%E5%9C%BA%E6%99%AF)
  - [ThreadLocal什么情况会出现OOM？](#threadlocal%E4%BB%80%E4%B9%88%E6%83%85%E5%86%B5%E4%BC%9A%E5%87%BA%E7%8E%B0oom)
  - [什么是双亲委派模型？有什么好处？什么情况下需要破坏双亲委派模型？](#%E4%BB%80%E4%B9%88%E6%98%AF%E5%8F%8C%E4%BA%B2%E5%A7%94%E6%B4%BE%E6%A8%A1%E5%9E%8B%E6%9C%89%E4%BB%80%E4%B9%88%E5%A5%BD%E5%A4%84%E4%BB%80%E4%B9%88%E6%83%85%E5%86%B5%E4%B8%8B%E9%9C%80%E8%A6%81%E7%A0%B4%E5%9D%8F%E5%8F%8C%E4%BA%B2%E5%A7%94%E6%B4%BE%E6%A8%A1%E5%9E%8B)
- [JVM](#jvm)
  - [GC分哪两种？minor Gc和full Gc有什么区别？什么时候触发full Gc？分别采用什么算法？](#gc%E5%88%86%E5%93%AA%E4%B8%A4%E7%A7%8Dminor-gc%E5%92%8Cfull-gc%E6%9C%89%E4%BB%80%E4%B9%88%E5%8C%BA%E5%88%AB%E4%BB%80%E4%B9%88%E6%97%B6%E5%80%99%E8%A7%A6%E5%8F%91full-gc%E5%88%86%E5%88%AB%E9%87%87%E7%94%A8%E4%BB%80%E4%B9%88%E7%AE%97%E6%B3%95)
  - [JVM有几种classload？为什么有这么多？](#jvm%E6%9C%89%E5%87%A0%E7%A7%8Dclassload%E4%B8%BA%E4%BB%80%E4%B9%88%E6%9C%89%E8%BF%99%E4%B9%88%E5%A4%9A)
- [Spring](#spring)
  - [SpringBoot的自动装配原理？](#springboot%E7%9A%84%E8%87%AA%E5%8A%A8%E8%A3%85%E9%85%8D%E5%8E%9F%E7%90%86)
- [消息队列](#%E6%B6%88%E6%81%AF%E9%98%9F%E5%88%97)
  - [消息中间件产品的优缺点？](#%E6%B6%88%E6%81%AF%E4%B8%AD%E9%97%B4%E4%BB%B6%E4%BA%A7%E5%93%81%E7%9A%84%E4%BC%98%E7%BC%BA%E7%82%B9)
  - [消息中间件如何保证一致性？ 如何进行消息的重试的机制？](#%E6%B6%88%E6%81%AF%E4%B8%AD%E9%97%B4%E4%BB%B6%E5%A6%82%E4%BD%95%E4%BF%9D%E8%AF%81%E4%B8%80%E8%87%B4%E6%80%A7-%E5%A6%82%E4%BD%95%E8%BF%9B%E8%A1%8C%E6%B6%88%E6%81%AF%E7%9A%84%E9%87%8D%E8%AF%95%E7%9A%84%E6%9C%BA%E5%88%B6)
- [Redis](#redis)
  - [Redis的数据类型及对应的数据结构](#redis%E7%9A%84%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B%E5%8F%8A%E5%AF%B9%E5%BA%94%E7%9A%84%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84)
- [分布式](#%E5%88%86%E5%B8%83%E5%BC%8F)
  - [分布式事务的理解？二阶段提交？三阶段提交？](#%E5%88%86%E5%B8%83%E5%BC%8F%E4%BA%8B%E5%8A%A1%E7%9A%84%E7%90%86%E8%A7%A3%E4%BA%8C%E9%98%B6%E6%AE%B5%E6%8F%90%E4%BA%A4%E4%B8%89%E9%98%B6%E6%AE%B5%E6%8F%90%E4%BA%A4)
- [Mysql](#mysql)
  - [锁机制介绍：行锁、表锁、共享锁、排他锁](#%E9%94%81%E6%9C%BA%E5%88%B6%E4%BB%8B%E7%BB%8D%E8%A1%8C%E9%94%81%E8%A1%A8%E9%94%81%E5%85%B1%E4%BA%AB%E9%94%81%E6%8E%92%E4%BB%96%E9%94%81)
  - [乐观锁的业务场景以及实现方式？](#%E4%B9%90%E8%A7%82%E9%94%81%E7%9A%84%E4%B8%9A%E5%8A%A1%E5%9C%BA%E6%99%AF%E4%BB%A5%E5%8F%8A%E5%AE%9E%E7%8E%B0%E6%96%B9%E5%BC%8F)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## 多线程

### ThreadLocal、volatile的实现原理和使用场景？

ThreadLocal是一个线程副本变量，存储了每个线程的变量副本，线程与线程之间独立互不影响

ThreadLocal有一个静态内部类，ThreadLocalMap，key是线程的引用

volatile是jvm的一个关键字，volatile主要有两个作用，一是保证了原子性，二是禁止指令重排

### ThreadLocal什么情况会出现OOM？

当线程一直没有销毁，而使用完没有手动清除ThreadLocal的强引用时，key和value不断增加，最终导致OOM

### 什么是双亲委派模型？有什么好处？什么情况下需要破坏双亲委派模型？

当类加载器收到一个类加载请求时，他并不是自己加载，而是交由他的父类加载器加载，依次递归，直到顶层类加载加载，顶层类能加载则直接返回，否则依次由其子类加载，
好处：避免类的重复加载，防止Java核心API被修改


---

## JVM

### GC分哪两种？minor Gc和full Gc有什么区别？什么时候触发full Gc？分别采用什么算法？

minor Gc：对新生代进行回收，一般也叫YGC

full Gc：对整个堆进行回收

垃圾回收算法：

- 标记清除
- 复制
- 标记整理
- 分代收集

新生代使用复制算法、标记清除，老年代使用标记整理、分代收集

gc堆分为新生代和老年代，当然每种收集器也不一样，像G1、ZGC这种新型的就是将堆划分为region区域，并没有新生代，老年代的固定区域的划分

**对象在jvm怎么分配的**

- 判断能否在栈上分配
- 判断是否是大对象，大对象直接分配在来老年代
- 判断是否可以在TLAB分配，TLAB是线程本地分配缓存，占用的是eden的空间

**触发minor gc和full gc的条件**

- minor gc 当eden区满时，触发minor gc
- full gc
    - 调用system.gc()时
    - 老年代空间不足时
    - 方法区空间不足
    - 由eden区、from Survivor向to Survivor区复制时，对象大小大于to Survivor大小，则把对象转到老年代，当对象大小大于老年代可用空间，触发full gc

### JVM有几种类加载器

- 启动类加载器
- 扩展类加载类
- 系统类加载器
- 自定义类加载器

---

## Spring

### SpringBoot的自动装配原理？

注解SpringBootApplication实际上是一个复合注解 主要有两个主要的注解： @AutoConfiguration和@AutoCompontScan

@AuotConfiguration注解会自动扫描MATE-INF下spring.factories文件，每个文件有个AutoConfig的配置类，通过加载这些自动类从而实现了spring的自动装配，@AutoCompontScan注解自动扫描包下面的组件注入到spring

## 消息队列

### 消息中间件产品的优缺点？

主流消息中间件：rabbitMQ，RocketMQ，

### 消息中间件如何保证一致性？ 如何进行消息的重试的机制？

- 强一致性
- 最终一致性
---

## Redis

### Redis的数据类型及对应的数据结构

- string
- hash
- list
- set
- zset
- bitmaps
- hyperLogLog
- geo

---

## 分布式

### 分布式事务的理解？二阶段提交？三阶段提交？

CAP理论

- 在一个分布式系统中，不可能同时满足一致性、可用性、分区容错性 BASE理论 基本可用、软状态、最终一致性

**二阶段提交**

- 一阶段

- 二阶段

**三阶段提交**

- 一阶段提交
- 二阶段提交
- 三阶段提交

---

## Mysql

### 锁机制介绍：行锁、表锁、共享锁、排他锁

- 行锁 对某一行数据进行
- 表锁 对整个表进行枷锁
- 共享锁 读读并行
- 排他锁 与其他锁互斥
- 间隙锁
- 自增锁

### 乐观锁的业务场景以及实现方式？

- 版本号机制
- CAS算法， 及比较并交换，有三个值，内存值V，旧的预期值A，新的预期值B，当且仅当内存值等于旧的预期值，将内存值替换为新的预期值B

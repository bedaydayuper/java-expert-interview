# Redis interview\(二\) Redis

## 1 Redis 简介

Remote Dictionary Server.

### 1.1 优点

1 速度快：数据在内存中。

> Redis 本质上是一个 Key-Value 类型的内存数据库，很像 Memcached ，整个数据库统统加载在内存当中进行操作，定期通过异步操作把数据库数据 flush 到硬盘上进行保存。
>
> 因为是纯内存操作，Redis 的性能非常出色，每秒可以处理超过 10 万次读写操作，是已知性能最快的 Key-Value 数据库。

2 支持丰富数据类型

5种基础类型，然后还有高级的数据结构。

3、丰富的特性

订阅与发布

key 过期策略

事务

计数  
。。。

4、持久化存储

提供RDB 和 AOF 两种。

5、高可用

* 内置Redis sentinel，提供高可用方案，实现主从故障自动转移
* 内置Redis cluster， 提供集群方案，实现基于槽的分片方案，从而支持更大的Redis规模。

### 1.2 缺点

1、Redis 是内存数据库，要受限于机器的内存大小

2、修改配置文件，进行重启，将硬盘中的数据加载进内存，时间比较久，这个过程中，Redis 不能提供服务。



## 2 Redis 线程模型

### 非阻塞IO,多路复用

> Redis 内部使用文件事件处理器 `file event handler`，这个文件事件处理器是单线程的，所以 Redis 才叫做单线程的模型。它采用 IO 多路复用机制同时监听多个 Socket，根据 Socket 上的事件来选择对应的事件处理器进行处理。

文件事件处理器的结构包含 4 个部分：

* 多个 Socket 。
* IO 多路复用程序。
* 文件事件分派器。
* 事件处理器（连接应答处理器、命令请求处理器、命令回复处理器）。



## 3 Redis单线程为什么这么快？



## 4 Redis 持久化



## 5 Redis 过去策略



## 6 Redis 淘汰策略



## 7 大key 处理



## 8 Redis 客户端



## 9 Redis 分布式锁



## 10 Redis pipeline



## 11 Redis 事务



## 12 Redis 集群方案？



## 13 Redis 主从同步



## 14 Redis sentinel 实现高可用



## 15 Redis cluster 实现高可用



## 16 Redis 健康指标



## 17 优化Redis 内存占用



## 18 Redis 常见性能问题与解决






---
description: 一些关于es 系统运维的操作指令。
---

# ES 日常操作汇总-运维

## 1 查看节点状态

### 1.1 查看单节点的shard分配整体情况 

```text
GET /_cat/allocation?v&pretty   

结果解析：
shards  分片
disk.indices  磁盘索引
disk.used    磁盘使用过的
disk.avail    磁盘有效
disk.total    磁盘总数
disk.percent   磁盘百分比
host    主机
ip    ip地址
node   节点

每个命令都支持使用?v参数，让输出内容表格显示表头; pretty则让输出缩进更规范
```

### 1.2 查看各shard 的详细情况

```text
GET /_cat/shards?v&pretty

结果举例：
index      shard prirep state      docs   store ip        node
bank       1     p      STARTED     191 122.8kb 127.0.0.1 es-node1
bank       1     r      UNASSIGNED                        

结果解析:
index  索引
shard 分片号
prirep  主分片还是副本  p/r 
status  状态
docs  文档数量
store  占用内存大小
```

如果是查看某个索引的分片情况，则使用如下指令：

```text
GET /_cat/shards/{index}
```



### 1.3 查看master 节点信息

```text
GET /_cat/master?v&pretty
结果举例：

id                     host      ip        node
zz-o14moS9aCm3O1gPFJYQ localhost 127.0.0.1 es-node1
```

### 1.4 查看所有节点信息

```text
GET /_cat/nodes?v&pretty 
结果举例：
ip        heap.percent ram.percent cpu load_1m load_5m load_15m node.role master name
127.0.0.1           27          87   8    2.43                  mdi       *      es-node1

结果字段分析：
heap.percent  堆使用比例
ram.percent memory（内存）使用比例
cpu cpu使用情况
load（平均负载） 1分钟，5分钟， 15 分钟 的负载

```

load 相关文章：https://www.jianshu.com/p/d07b51fe9855  


> load average 表示的是CPU的负载，包含的信息不是CPU的使用率状况，而是在一段时间内CPU正在处理以及等待CPU处理的进程数之和的统计信息，也就是CPU使用队列的长度的统计信息。

### 1.5 查看索引的信息

```text
GET /_cat/indices?v&pretty

结果举例：
health status index      uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   bank       8SE_rI-AQc2wKiDF4DW1lA   5   1       1000            0    640.6kb        640.6kb
yellow open   .kibana    xw3cTjI1T5u6hltILoAwlQ   1   1          1            0      3.1kb          3.1kb

结果分析：
pri 主分片数量
rep 副本数量
docs.count 文档数量
docs.deleted 删除文档数量
store.size 存储占用大小
```

如果要查看某个索引的信息，可以使用如下指令：

```text
GET _cat/indices/{index}
```

### 1.6 查看分片信息

```text
GET /_cat/segments?v&pretty

结果举例：
index      shard prirep ip        segment generation docs.count docs.deleted    size size.memory committed searchable version compound
bank       0     p      127.0.0.1 _0               0        197            0 126.2kb       10511 true      true       6.5.0   true
bank       1     p      127.0.0.1 _0               0        191            0 122.6kb       10296 true      true       6.5.0   true
bank       2     p      127.0.0.1 _0               0        211            0 134.1kb       10818 true      true       6.5.0   true
bank       3     p      127.0.0.1 _0               0        200            0 127.9kb       10542 true      true       6.5.0   true
bank       4     p      127.0.0.1 _0               0        201            0 128.5kb       10590 true      true       6.5.0   true


结果分析：
http://doc.codingdict.com/elasticsearch/222/
size 磁盘大小
size.memory  相关内容占用内存的大小
committed  是否已经落到磁盘上了。
compound   底层Lucene 是否已经将多个文件合并为一个文件
```

### 1.7 查看文档数量

```text
GET /_cat/count?v&pretty 

结果举例：
epoch      timestamp count
1613272429 11:13:49  1010

结果分析：
```

### 1.8 集群内不同类型线程池的统计信息

```text
GET /_cat/thread_pool?v&pretty 
结果举例：
node_name name                active queue rejected
es-node1  bulk                     0     0        0
es-node1  fetch_shard_started      0     0        0
es-node1  fetch_shard_store        0     0        0
es-node1  flush                    0     0        0
es-node1  force_merge              0     0        0
es-node1  generic                  0     0        0
es-node1  get                      0     0        0
es-node1  index                    0     0        0
es-node1  listener                 0     0        0
es-node1  management               1     0        0
es-node1  refresh                  0     0        0
es-node1  search                   0     0        0
es-node1  snapshot                 0     0        0
es-node1  warmer                   0     0        0

结果分析：
node_name 节点名
name  线程池名称
active（活跃的），queue（队列中的）和 reject（拒绝的）的统计信息

```

## 参考文献：

### [https://www.cnblogs.com/qdhxhz/p/11461174.html](https://www.cnblogs.com/qdhxhz/p/11461174.html)  


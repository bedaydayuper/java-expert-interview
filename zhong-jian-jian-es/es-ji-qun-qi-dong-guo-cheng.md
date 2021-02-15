# ES 集群启动过程

## 1 启动流程

![&#x542F;&#x52A8;&#x6D41;&#x7A0B;](../.gitbook/assets/image%20%2821%29.png)

## 2 确定主节点

对节点ID 排序，谁大谁是master。

基于 节点排序的简单选举有三个附加的约束条件：

* 参选人数过半
* 得票人数过半
* 当节点离开时，必须判断当前节点数是否过半。 
* 
![&#x9009;&#x4E3B;&#x5168;&#x6D41;&#x7A0B;](../.gitbook/assets/image%20%2815%29.png)

![&#x9009;&#x4E3E;&#x4E34;&#x65F6;master &#x6D41;&#x7A0B;](../.gitbook/assets/image%20%2826%29.png)

![&#x786E;&#x7ACB;master](../.gitbook/assets/image%20%2817%29.png)

![&#x5B9A;&#x671F;&#x68C0;&#x6D4B;](../.gitbook/assets/image%20%2818%29.png)

## 3 选举元信息

3.1 读写并不经过 master 节点，所以master节点需要收集元信息。

3.2 集群元信息（一些状态信息）选举之后，master 发布首次集群状态，然后开始选举shard 级元信息。

## 4 确定主分片

3.3 选举shard级的元信息，构建内容路由表

包括  主分片和副分片。  


主分片的确定由master节点决定。



## 5 主/ 副分片恢复

### 5.1 主分片恢复

借助事务日志，进行重放，如此完成主分片的恢复。

### 5.2 副分片恢复

划分了两个阶段：

![](../.gitbook/assets/image%20%2816%29.png)


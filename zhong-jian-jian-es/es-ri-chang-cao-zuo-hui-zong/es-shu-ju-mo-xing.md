# ES 数据模型

## 1 数据副本策略

![&#x6570;&#x636E;&#x526F;&#x672C;&#x7B56;&#x7565;](../../.gitbook/assets/image%20%2811%29.png)

## 2 ES 数据副本模型

### 2.1 基本写入模型

![](../../.gitbook/assets/image%20%2819%29.png)

### 2.2 基本读取模型

![](../../.gitbook/assets/image%20%2813%29.png)

## 3 Allocation Id

用于主节点决定哪个分片应该分配到哪个节点，以及哪个分片作为主分片。

主节点会维护一个最新shard 的Application id 集合，这个集合维护了最新的数据，只有在这个集合中的分片，才有资格被选为主分片。

## 4 Sequence Id 





## 5 


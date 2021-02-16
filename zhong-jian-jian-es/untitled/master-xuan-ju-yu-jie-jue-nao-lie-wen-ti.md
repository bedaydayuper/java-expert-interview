# master  选举与解决脑裂问题

## 1 master 选举

前置条件：

1. 只有是候选主节点（master：true）的节点才能成为主节点。
2. 最小主节点数（min\_master\_nodes）的目的是防止脑裂。

 

选举流程大致描述如下：

1. 确认候选主节点数达标，elasticsearch.yml 设置的值 discovery.zen.minimum\_master\_nodes; 
2. 对所有候选主节点根据nodeId字典排序，每次选举每个节点都把自己所知道节点排一次序，然后选出第一个（第0位）节点，暂且认为它是master节点。\`临时master\`
3. 如果对某个节点的投票数达到一定的值（候选主节点数n/2+1）并且该节点自己也选举自己，那这个节点就是master。否则重新选举一直到满足上述条件。

## 2 解决脑裂问题 

> 所谓集群脑裂，是指 Elasticsearch 集群中的节点（比如共 20 个），其中的 10 个选了一个 master，另外 10 个选了另一个 master 的情况。

有两种情况：

1. 当集群 master 候选数量不小于 3 个时，可以通过设置最少投票通过数量（discovery.zen.minimum\_master\_nodes）超过所有候选节点一半以上来解决脑裂问题；
2. 当候选数量为两个时，只能修改为唯一的一个 master 候选，其他作为 data 节点，避免脑裂问题。

## 参考文献：

[https://www.wenyuanblog.com/blogs/elasticsearch-interview-questions.html](https://www.wenyuanblog.com/blogs/elasticsearch-interview-questions.html)


# ES 面试题（二）集群管理与读写流程

## 1 集群启动

### 1.1 启动流程

![&#x542F;&#x52A8;](../../.gitbook/assets/image%20%2846%29.png)

### 1.2 确定主节点

对节点ID 排序，谁大谁是master。

基于 节点排序的简单选举有三个附加的约束条件：

* 参选人数过半
* 得票人数过半
* 当节点离开时，必须判断当前节点数是否过半。

![&#x9009;&#x4E3B;&#x5168;&#x6D41;&#x7A0B;](../../.gitbook/assets/image%20%2839%29.png)

![&#x9009;&#x4E3E;&#x4E34;&#x65F6;master](../../.gitbook/assets/image%20%2842%29.png)

![&#x786E;&#x7ACB;master](../../.gitbook/assets/image%20%2838%29.png)

![&#x5B9A;&#x671F;&#x68C0;&#x67E5;](../../.gitbook/assets/image%20%2848%29.png)

### 1.3 选举元信息

1.3.1 读写并不经过 master 节点，所以master节点需要收集元信息。

1.3.2 集群元信息（一些状态信息）选举之后，master 发布首次集群状态，然后开始选举shard 级元信息。

### 1.4 确定主分片

选举shard级的元信息，构建内容路由表

包括  主分片和副分片。

主分片的确定由master节点决定。

### 1.5 主副分片恢复

#### 1.5.1 主分片恢复

借助事务日志，进行重放，如此完成主分片的恢复。

#### 1.5.2 副分片恢复

划分了两个阶段：  


![](../../.gitbook/assets/image%20%2843%29.png)



## 2 单个节点的启动与关闭

![&#x8282;&#x70B9;&#x542F;&#x52A8;](../../.gitbook/assets/image%20%2835%29.png)

![&#x8282;&#x70B9;&#x5173;&#x95ED;](../../.gitbook/assets/image%20%2845%29.png)

## 3倒排索引

es 中不仅仅建立了 ”term dictionary -- doc “倒排索引，而且还对词进行了索引（term index -- term dictionary）,这样方便快速进行找到词，然后再由词找到对应的文档。

并且底层是基于FST\(Finite State Transducer\) 数据结构，对词典中的单词前缀和后缀进行重复利用，空间占用小。进而减少了访问磁盘的次数，提高了性能。

## 4 读写

### 4.1 写过程

![](../../.gitbook/assets/image%20%2836%29.png)



> 写过程：  
> client 发送请求到协调节点，协调节点通过hash ，找到对应的主分片节点，然后转发到所有主分片所在的节点，然后主分片节点顺序写每条数据，然后并行转发到各个副分片节点，等到所有副分片节点返回之后，再响应到主分片节点。当所有主分片节点 收到各自所有的副分片之后，再响应协调节点。最后协调节点响应给客户端。

### 4.2 get 过程

![](../../.gitbook/assets/image%20%2844%29.png)



get 请求的过程：

> client 请求发送给 协调节点，然后协调节点通过id 进行hash ,获取对应数据节点（数据节点可以是主分片节点，也可以是副分片节点），然后将请求转发给数据节点，最后等到数据节点响应之后，然后协调节点响应给client 。

### 4.3 search 过程

![](../../.gitbook/assets/image%20%2841%29.png)



search 过程：

> 客户端发送请求给协调节点，协调节点将搜索请求转发到`所有`  
>  的shard 节点对应的primary shard 或者 replica shard 。
>
> query phase：  每个shard 将自己的搜索结果（其实就是一些doc id）返回给协调节点，由协调节点进行数据的合并、排序、分页等操作，产生最终数据。
>
> fetch phase: 接着由协调节点根据doc id 去各个节点拉取实际的document 数据，最终返回给客户端。

### 4.4 单台机器上写操作-底层原理

![](../../.gitbook/assets/image%20%2840%29.png)



> 1. 先写入buffer，在buffer里的时候数据是搜索不到的；同时将数据写入translog日志文件
> 2. 如果buffer快满了，或者每隔一秒钟，就会将buffer数据refresh到一个新的segment file中并清空buffer，但是此时数据不是直接进入segment file的磁盘文件的，而是先进入os cache的。当数据进入os cache后，就代表该数据可以被检索到了。因此说es是准实时的，这个过程就是**refresh**。
> 3. 只要数据进入os cache，此时就可以让这个segment file的数据对外提供搜索了
> 4. 重复1~3步骤，新的数据不断进入buffer和translog，不断将buffer数据写入一个又一个新的segment file中去，每次refresh完buffer清空，translog保留。随着这个过程推进，translog会变得越来越大。当translog达到一定长度的时候，就会触发commit操作。  
>
>
>    **commit**操作（也叫**flush**操作，默认每隔30分钟执行一次）：执行refresh操作 -&gt; 写commit point -&gt; 将os cache数据fsync强刷到磁盘上去 -&gt; 清空translog日志文件
>
>    commit操作保证了在机器宕机时，buffer和os cache中未同步到segment file中的数据还可以在重启之后恢复到内存buffer和os cache中去，
>
> 5. translog其实也是先写入os cache的，默认每隔5秒刷一次到磁盘中去，所以默认情况下，可能有5秒的数据会仅仅停留在buffer或者translog文件的os cache中，如果此时机器挂了，会丢失5秒钟的数据。但是这样性能比较好，最多丢5秒的数据。也可以将translog设置成每次写操作必须是直接fsync到磁盘，但是性能会差很多。
> 6. 如果是删除操作，commit的时候会生成一个.del文件，里面将某个doc标识为deleted状态，那么搜索的时候根据.del文件就知道这个doc被删除了
> 7. 如果是更新操作，就是将原来的doc标识为deleted状态，然后新写入一条数据
> 8. buffer每次refresh一次，就会产生一个segment file，所以默认情况下是1秒钟一个segment file，segment file会越来越多，此时会定期执行merge,当segment多到一定的程度时，自动触发merge操作
> 9. 每次**merge**的时候，会将多个segment file合并成一个，同时这里会将标识为deleted的doc给物理删除掉，然后将新的segment file写入磁盘，这里会写一个commit point，标识所有新的segment file，然后打开segment file供搜索使用，同时删除旧的segment file。
>
>   
>   
>   
> 注意： translog 记录会每次执行写操作理解写入os cache, 然后 5s 写入一次磁盘，因为如果每次translog 记录 都直接写入磁盘，性能会非常差。

附属问题1： refresh和flush有什么区别

refresh 是从buffer 刷新到oscache

而 flush 是从os cache 刷新到磁盘。

附属问题2： translog日志有什么作用

记录每次的写操作，用于恢复数据。

附属问题3： translog文件和segment文件有什么区别

translog 是记录操作

segment 是doc 本身，记录了内容。

### 4.5 删除/更新数据-底层原理

#### 4.5.1 删除

如果是删除操作，commit 的时候会生成一个.del 文件，里面将某个doc 标识为deleted 状态，那么搜索的时候根据.del 文件就知道这个doc 是否被删除了。

#### 4.5.2 update

将原来的文件标记为deleted 状态，然后新写入一条数据。

#### 4.5.3 merge 与 物理删除

> buffer 每 refresh 一次，就会产生一个 `segment file` ，所以默认情况下是 1 秒钟一个 `segment file` ，这样下来 `segment file` 会越来越多，此时会定期执行 merge。每次 merge 的时候，会将多个 `segment file` 合并成一个，同时这里会将标识为 `deleted` 的 doc 给**物理删除掉**，然后将新的 `segment file` 写入磁盘，这里会写一个 `commit point` ，标识所有新的 `segment file` ，然后打开 `segment file` 供搜索使用，同时删除旧的 `segment file` 。

#### 4.5.4 索引生命周期

索引是在写入时创建，创建之后不可变。在更新或者删除时，被标记为 deleted 状态，在segment merge 时，进行物理删除。





## 5 master 选举

前置条件：

1. 只有是候选主节点（master：true）的节点才能成为主节点。
2. 最小主节点数（min\_master\_nodes）的目的是防止脑裂。

选举流程大致描述如下：

1. 确认候选主节点数达标，elasticsearch.yml 设置的值 discovery.zen.minimum\_master\_nodes; 
2. 对所有候选主节点根据nodeId字典排序，每次选举每个节点都把自己所知道节点排一次序，然后选出第一个（第0位）节点，暂且认为它是master节点。\`临时master\`
3. 如果对某个节点的投票数达到一定的值（候选主节点数n/2+1）并且该节点自己也选举自己，那这个节点就是master。否则重新选举一直到满足上述条件。

## 6 脑裂问题 

> 所谓集群脑裂，是指 Elasticsearch 集群中的节点（比如共 20 个），其中的 10 个选了一个 master，另外 10 个选了另一个 master 的情况。

有两种情况：

1. 当集群 master 候选数量不小于 3 个时，可以通过设置最少投票通过数量（discovery.zen.minimum\_master\_nodes）超过所有候选节点一半以上来解决脑裂问题；
2. 当候选数量为两个时，只能修改为唯一的一个 master 候选，其他作为 data 节点，避免脑裂问题。

> 一般生产环境，都要求至少3个节点。

## 参考文献：

[https://www.javazhiyin.com/66545.html](https://www.javazhiyin.com/66545.html)  
[https://www.wenyuanblog.com/blogs/elasticsearch-interview-questions.html](https://www.wenyuanblog.com/blogs/elasticsearch-interview-questions.html)




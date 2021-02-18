# ES 读写与search 流程, 以及底层原理

## 1 写过程

![&#x5199;&#x6D41;&#x7A0B;](../../.gitbook/assets/image%20%2811%29.png)

> 写过程：  
> client 发送请求到协调节点，协调节点通过hash ，找到对应的主分片节点，然后转发到所有主分片所在的节点，然后主分片节点顺序写每条数据，然后并行转发到各个副分片节点，等到所有副分片节点返回之后，再响应到主分片节点。当所有主分片节点 收到各自所有的副分片之后，再响应协调节点。最后协调节点响应给客户端。

## 2 get 过程

![GET &#x8BF7;&#x6C42;](../../.gitbook/assets/image%20%2820%29.png)

get 请求的过程：

> client 请求发送给 协调节点，然后协调节点通过id 进行hash ,获取对应数据节点（数据节点可以是主分片节点，也可以是副分片节点），然后将请求转发给数据节点，最后等到数据节点响应之后，然后协调节点响应给client 。

## 3 search 过程

![search](../../.gitbook/assets/image%20%2813%29.png)

search 过程：

> 客户端发送请求给协调节点，协调节点将搜索请求转发到`所有`  
>  的shard 节点对应的primary shard 或者 replica shard 。
>
> query phase：  每个shard 将自己的搜索结果（其实就是一些doc id）返回给协调节点，由协调节点进行数据的合并、排序、分页等操作，产生最终数据。
>
> fetch phase: 接着由协调节点根据doc id 去各个节点拉取实际的document 数据，最终返回给客户端。



## 4 单台机器上的写操作底层原理

![&#x5199;&#x64CD;&#x4F5C;&#x5E95;&#x5C42;&#x539F;&#x7406;](../../.gitbook/assets/image%20%2823%29.png)

![](../../.gitbook/assets/image%20%2834%29.png)

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

### 4.2 refresh和flush有什么区别

refresh 是从buffer 刷新到oscache

而 flush 是从os cache 刷新到磁盘。

### 4.3 translog日志有什么作用

记录每次的写操作，用于恢复数据。

### 4.4 translog文件和segment文件有什么区别

translog 是记录操作

segment 是doc 本身，记录了内容。

## 5 删除/ 更新数据底层原理

### 5.1 删除

如果是删除操作，commit 的时候会生成一个.del 文件，里面将某个doc 标识为deleted 状态，那么搜索的时候根据.del 文件就知道这个doc 是否被删除了。

### 5.2 update

将原来的文件标记为deleted 状态，然后新写入一条数据。

### 5.3 merge 与 物理删除

> buffer 每 refresh 一次，就会产生一个 `segment file` ，所以默认情况下是 1 秒钟一个 `segment file` ，这样下来 `segment file` 会越来越多，此时会定期执行 merge。每次 merge 的时候，会将多个 `segment file` 合并成一个，同时这里会将标识为 `deleted` 的 doc 给**物理删除掉**，然后将新的 `segment file` 写入磁盘，这里会写一个 `commit point` ，标识所有新的 `segment file` ，然后打开 `segment file` 供搜索使用，同时删除旧的 `segment file` 。

### 5.4 索引生命周期

索引是在写入时创建，创建之后不可变。在更新或者删除时，被标记为 deleted 状态，在segment merge 时，进行物理删除。

## 参考

[https://www.javazhiyin.com/66545.html](https://www.javazhiyin.com/66545.html)




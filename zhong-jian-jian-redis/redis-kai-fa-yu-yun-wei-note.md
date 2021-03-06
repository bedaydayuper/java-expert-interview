---
description: 当当电子书
---

# Redis开发与运维-note（一 API操作）

## 1 初识Redis

1、常见的数据结构：字符串、哈希、列表、集合、有序集合。 然后在字符串的基础上演变出了位图、hyperLogLog ，并且随着LBS 的发展，加入了GEO\(地理信息定位\)的功能。

2、基础数据操作之外，还有如下额外的功能：

* 键过期功能，可以实现缓存
* 发布订阅功能，实现消息系统
* 支持Lua 脚本功能，利用Lua 创造出新的Redis 命令
* 简单的事务特性
* 流水线（pipeline）功能

3、持久化 RDB 和 AOF

4、复制功能，实现了多个相同数据的Redis副本。

5、提供了Redis sentinel，能够保证Redis节点的故障发现和故障自动转移。



## 2  API 的理解和使用

### 2.1 预备

1、查看所有键：需要遍历所有的键，O\(n\), 所以当保存了大量键时，线上环境禁止使用。

```text
keys *
```

2、键总数：

```text
dbsize
```

3、检查键是否存在：

```text
exists key
```

4、删除键

```text
del key

del key1 key2 ...
```

5、键过期

```text
expire key seconds
```

查看剩余时间：ttl

```text
ttl key
```

6、查看key 的数据结构类型\(常见的对外五种：string， hash, list, set, zset\)

```text
type key
```

![5&#x79CD;&#x5BF9;&#x5916;&#x5448;&#x73B0;&#x7684;&#x6570;&#x636E;&#x7ED3;&#x6784;](../.gitbook/assets/image%20%2879%29.png)

7、查看key 的内部编码

```text
object encoding key 
```

![&#x6570;&#x636E;&#x7ED3;&#x6784;&#x4E0E;&#x5185;&#x90E8;&#x7F16;&#x7801;](../.gitbook/assets/image%20%2874%29.png)

8、单线程架构  
\`单线程 + IO多路复用 = 高性能\`

所有的客户端请求，到达服务端之后，都需要先进入队列，进行排队。不存在被同时执行的情况。

 9、单线程每秒万级别的处理能力，原因是：

* 1、纯内存访问 
* 2、非阻塞I/O, 使用epoll 作为I/O 多路复用技术的实现，不在网络I/O上浪费过多的时间。 
* 3、单线程避免了线程切换和竞态产生的消耗

单线程的缺陷：

> 如果某个命令的执行时间过长，会造成其他命令的阻塞。



### 2.2 字符串

1、字符串类型的实际值可以是字符串、数字、二进制等等。最大不超过512M。

2、设置：

```text
set key value [ex seconds] [px milliseconds] [nx|xx]

解释：
ex： 秒级过期时间
px: 毫秒级过期时间
nx: 键必须不存在，才可以设置成功，用于添加
xx: 键必须存在，才可以设置成功，用于更新。

```

setnx: 如果多个客户端同时执行setnx key value, 只能有一个客户端成功，所以可以作为分布式锁的一种实现方案。

3、批量设置与获取：

```text
mset key value [key value ...]

mget key1 key2 ...
```

> 学会使用批量操作，有助于提高业务处理效率，但是注意，批量发送的命令数不是无节制的，如果数量过多可能会造成Redis 阻塞。

4、计数：  
incr 对值做自增操作。

* 值不是整数，返回错误
* 值是整数，返回自增后的结果
* 键不存在，按照值为0自增，返回结果为1.

```text
incr key

decr key 自减

incrby key number  按照数字自增

decrby key number 按照数字自减
```

5、内部编码

字符串类型的内部编码有3种：

* int  8个字节的长整型
* embstr 小于等于39个字节的字符串
* raw 大于39个字节的字符串

之所以这么区分，是为了更好地进行内存优化。

### 2.3 哈希

1、value 本身又是一个键值对，形如  

```text
value={{field1, value1}, ..., {fieldN, valueN}}
```

![](../.gitbook/assets/image%20%2876%29.png)

2、基础操作

```text
// 设置
hset key field value
eg:
hset user:1 name zp

// 获取
hget key field
eg
hget user:1 name

// 删除field
hdel key field [field...]
eg:
hdel user:1 name 

// 计算field 个数
hlen key
eg:
hlen user:1  // 返回具体的个数

// 批量设置或者获取 field-value 
hmget key field [field...]
hmset key field value [field value ...]

// 判断field 是否存在
hexists key field

// 获取所有的field
hkeys key
eg:
hkeys user:1
// 获取所有的value
hvals key
eg:
hvals user:1

// 获取所有的field-value
hgetall key
eg:
hgetall user:1

// 增加field 的值
hincrby key field
eg:
localhost:6379> hset user:1 age 1
(integer) 1
localhost:6379> hincrby user:1 age 3
(integer) 4
localhost:6379> hget user:1 age
"4"

```

> 使用hgetall 时，如果哈希元素个数比较多，会存在阻塞Redis 的可能。如果想获取部分field， 可以使用hmget , 如果一定要获取全部field-value, 可以使用hscan 命令，该命令会渐进式遍历哈希类型。



3、内部编码

* ziplist: 压缩列表：当哈希类型元素个数小于hash-max-ziplist-entries\(默认512个\)，同时所有值都小于hash-max-ziplist-value（默认64字节）时，会使用ziplist, 节省内存。
* hashtable: 哈希表，不满足ziplist 时，使用hashtable.

### 2.4 列表

1、用来存储多个有序的字符串，每个字符串成为元素，一个列表最多存储2^32 -1 个元素。在Redis中，可以对列表两端插入和弹出，还可以获取指定范围的元素列表、获取指定索引下标的元素等。可以充当栈和队列的角色。

2、列表类型的特点

* 列表中的元素是有序的
* 列表中的元素是可以重复的。

3、命令

```text
//1 添加
// 从右边插入
rpush key value [value...]
// 从左边插入
lpush key value [value ...]
// 向某个元素前或者后插入元素: 先找到元素pivot, 然后在其前或者后插入新元素 value
linsert key before|after pivot value
eg:
linsert testkey before b java


//2 查询
// 获取指定范围内的元素列表
lrange key start end 
索引下标 特点：
第一、索引下标从左到右分别是0到N-1, 但是从右到左 分别是 -1 到 -N.
第二、lrange 中的end 选项包含了自身。

// 获取指定索引下标的元素
lindex key index
eg:
lindex testkey -1 // 最后一个元素

// 获取长度
llen key 

//3 删除
// 从列表左侧弹出元素
lpop key
// 从列表右侧弹出元素
rpop key 
// 删除指定元素
lrem key count value  
从列表中找到等于value 的元素删掉，根据count 的不同分为三种情况：
（1）count > 0, 从左到右，删除最多count 个元素
（2）count < 0, 从右到左，删除最多count 绝对值个元素
（3）count=0 删除所有。

// 按照索引范围修建列表
ltrim key start end
比如
ltrim testkey 1 3 // 只保留testkey 第2个到第4个元素。

localhost:6379> lpush testkey 1 2 3 4
(integer) 4
localhost:6379> lrange testkey 0 -1
1) "4"
2) "3"
3) "2"
4) "1"
localhost:6379> ltrim testkey 2 3
OK
localhost:6379> lrange testkey 0 -1
1) "2"
2) "1"

//4 修改
lset key index newValue


//5 阻塞操作
blpop 和 brpop 是 lpop 和 rpop的阻塞版本。
brpop key timeout.  // timeout 单位为秒，客户端要等到timeout 秒后返回，如果设置为0，则会一直阻塞。

```

4、内部编码

* ziplist: 当列表的元素个数小于list-max-ziplist-entries ,同时列表中每个元素都小于list-max-ziplist-value 配置时，使用这种数据结构。
* linkedlist：无法满足ziplist , 则使用linkedlist。 

5、适用场景

![](../.gitbook/assets/image%20%2873%29.png)

### 2.5 集合

1、不允许有重复，并且无序，也不能通过下标获取元素。

2、命令：

```text
// 添加
sadd key element [element...]

// 删除
srem key elment [element...]

// 计算个数
scard key

// 判断元素是否在集合中
 sismember key element // 有，返回1； 没有返回0.
 
// 从集合返回指定个数元素
srandmember key [count] 

// 从集合中随机弹出元素
spop key

 // 获取所有的元素
smembers key

// 集合间操作-交集
sinter key [key...]

 // 集合间操作-并集
 sunion key [key ...]
 // 集合间差集
  sdiff key [key...]
```

3、内部编码

* intset\(整数集合\)：当元素中的元素都是整数，而且元素个数小于set-max-intset-entries 配置时，使用。
* hashtable\(哈希表\): 无法满足intset 时使用。

### 2.6 有序集合

1、首先是集合，所以不能有重复的元素（但是分数可以重复），但不同的是有序集合是可以排序的。给每个元素设置一个分数作为排序的依据。提供了获取制定分数和元素范围查询、计算成员排名等功能。

![](../.gitbook/assets/image%20%2877%29.png)

  
2、命令

```text
// 添加元素
zadd key score member [score member ...]

// 计算成员个数
zcard key
// 返回指定分数范围成员个数
zcount key min max  
min 和max 为开始分数和结束分数


// 计算某个成员的分数
zscore key member

// 计算成员的排名
zrank key member  // 从低到高排名
zrevrank key member // 从高到低排名

// 删除成员
zrem key member [key member ...]

// 增加成员的分数
zincrby key increment member
eg:
zincrby userkey 9 zp // 给 userkey 中 member 为 zp 的元素 增加分数 9 。

// 返回指定排名返回的成员
zrange key start end [withscores]  // 从低到高 返回排名在start 和 end 之间的元素，带上withscore ,则返回值带有分数。
zrevrange key start end [withscores] // 从高到低
eg:
zrange testkey 0 2 withscores 返回排名最低的三个成员

// 返回指定分数范围的成员
zrangebyscore key min max [withscores] [limit offset count]
zrevrangebyscore key max min [withscores] [limit offset count]

其中：
key 为zset的key
min 和max 为最小分数 和最大分数
withscores 表示结果返回是否带score
limit offset count ： 表示限制输出的起始位置和个数

// 删除指定排名内的升序元素 (注意是排名)
zremrangebyrank key start end 
eg: 
zremrangebyrank testkey 0 2 // 删除第start名 到第end 名的元素

// 删除指定分数内的成员
zremrangebyscore key min max // min 和max 为分数
eg:
zremrangebyscore testkey (250 +inf  // 删除250分以上的全部成员

// 集合间的命令 [不常用，忽略]

```

3、内部编码  


* ziplist\(压缩列表\): 当有序集合的元素个数小于zset-max-ziplist-entries ，且每个元素的值都小于zset-max-ziplist-value 配置时，使用。
* skiplist:ziplist 不满足的情况。

### 2.7 键管理

1、单个键管理

```text
// 键重命名
rename key newkey
renamenx key newkey // 只有newkey 不存在时才会被覆盖

// 随机返回一个键
randomkey

// 键过期
expire key seconds // 键在多少秒之后过期
ttl key // 查看key 的过期时间还剩多少。 
ttl 结果：
大于0，是还剩多少秒； 
-1 表示没有设置过期时间； 
-2 表示键不存在。

Redis不支持二级数据结构（比如哈希、列表）内部元素的过期。

setex 作为set+expire 的组合，不但是原子操作，还减少了一次网络通讯的时间。

// 迁移键
(1)方案一：生产环境一般不允许
move key db  // 把指定的键从源数据库移动到目标数据库中。
（2）方案二：dump + restore: 
dump key // 将键值序列化
restore key ttl value // 在目标Redis上，复原。
（3）方案三、migrate



```

2、遍历键  
两种方案：keys 和scan

\(1\) keys 全量遍历键

```text
keys pattern // pattern 为正则表达式
```

如果Redis 里面包含了大量键，执行keys 命令很可能会造成Redis阻塞，所以生产环境一般不用。

（2）scan 渐进式遍历

scan 采用渐进式遍历的方式来解决keys 命令可能带来的阻塞问题，每次scan 命令的时间复杂度为O\(1\), 但是要真正实现keys 的功能，需要执行多次scan 。

```text
scan cursor [match pattern] [count number]
其中：
cursor是必须参数，游标，第一次遍历从0开始，每次scan 遍历都会返回当前游标的值，直到游标值为0，表示遍历结束。
match pattern: 可选，做模式的匹配
count number: 可选参数：作用是表明每次要遍历的键的个数，默认是10，此参数可以适当调大。
```

除了scan 以外，Redis提供了面向哈希、集合、有序集合的扫描遍历命令，解决诸如hgetall, smembers, zrange 可能产生的阻塞问题，对应的命令分别是hscan, sscan, zscan。

> 渐进式遍历可以解决keys 可能产生的阻塞问题，但是scan 并非完美，如果在scan 的过程中如果有键的变化（增加、删除、修改），那么遍历效果可能会碰到如下问题：新增的键可能没有遍历到，遍历出了重复的键等情况，也就是说scan 并不能保证完整的遍历出来所有的键。






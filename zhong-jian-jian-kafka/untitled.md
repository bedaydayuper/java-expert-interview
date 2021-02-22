# 深入理解kafka

## 1 初始kafka

1、主题是一个逻辑上的概念，可以细分为多个分区。

同一主题下的不同分区包含的消息是不同的。

kafka 保证的是分区有序而不是主题有序。

2、leader 副本 负责读与写，而replica 副本 值负责与leader 副本的消息同步。replica 是用于容灾。  


3、

![](../.gitbook/assets/image%20%2819%29.png)





## 2 生产者

1、消息对象的结构

```text
public class ProducerRecord<K, V> {
    private final String topic; // 主题
    private final integer partition; // 分区号
    private final Headers headers; // 消息头部
    private final K key; // 键
    private final V value; // 值
    private final Long timestamp; // 消息时间戳
    // 省略其他成员方法和构造方法
}
```

key: 可以用来指定消息的键，不进是消息的附加信息，还可以用来计算分区号，进而可以让消息发往特定的分区。同一个key 的消息会被划分到同一个分区中。



2、构建生产者

构建生产者，至少需要如下参数：

```text
bootstrap.servers:  broker 地址清单
key.serializer 和 value.serializer: broker 端接收的消息必须以字节数组的形式存在。
在发往broker 之前，都要使用序列化器进行转换成字节数组。
看了一下 org.apache.kafka.common.serialization.StringSerializer, 就是一个StringCoding.encode.


```

3、消息体构建 ProducerRecord

topic

value

Header

4、发送的类型

发后即忘： 消息丢失  
同步：需要等待，性能差  
异步：可以获取其对应的结果，也可以不捕获



5、分区器

在发往broker的过程中，如果没有指定partition字段，那么就需要依赖分区器。分区器需要依赖消息对象 productRecord 的key 字段、主题、值、以及集群的元数据信息。

> 如果key 不为null, 那么计算得到的分区号会是所有分区中的任意一个； 如果key 为 null, 那么计算得到的分区号仅为可用分区中的任意一个。

消息在分区内是有序的，如果某个业务需要按照某个规则有序，则可以自定义分区器。比如商品属于某个仓库，则存储商品时，可以按照商品的仓库信息进行 分区。

6、拦截器

拦截器分为生产者拦截器和消费者拦截器。

生产者拦截器既可以用来在消息发送前做一些准备工作，比如过滤消息，修改消息内容，也可以用来在发送回调逻辑前做一些定制化的需求。

7、整体架构

![](../.gitbook/assets/image%20%2850%29.png)

整个发送过程由主线程 和 sender 线程协调运行。

> 主线程负责创建消息，然后通过拦截器、序列化器和分区器的作用之后，缓存到消息累加器；

> sender 线程负责从消息累加器中获取消息并将其发送到kafka 中。

消息累加器 RecordAccumulator 用来缓存消息以便sender 线程可以批量发送，进而减少网络的资源消耗。

元数据：是指kafka集群的元数据, 这些元数据具体记录了集群中有哪些主题，这些主题有哪些分区，每个分区的leader 副本分配在哪个节点上，follower  副本分配在哪些节点上，哪些副本在AR、ISR 等集合中，集群中有哪些节点，控制器节点又是哪一个等信息。

Sender 线程会定期获取具体的元信息数据。



  
























## 3 消费者



## 4 主题与分区



## 5 日志存储



## 6 深入服务端



## 7 深入客户端



## 8 可靠性研究



## 9 高级应用 （11章）


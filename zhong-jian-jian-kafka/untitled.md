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



3、消息体构建 ProducerRecord



4、发送的类型

发后即忘： 消息丢失  
同步：需要等待，性能差  
异步：可以获取其对应的结果，也可以不捕获









## 3 消费者



## 4 主题与分区



## 5 日志存储



## 6 深入服务端



## 7 深入客户端



## 8 可靠性研究



## 9 高级应用 （11章）


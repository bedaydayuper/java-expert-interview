# kafka-面试题

## 1 消息队列的用途？

主要用于应用解耦、异步消息、流量削锋、消息通讯、日志处理等问题。

### 1.1 应用解耦

传统模式：

![](../.gitbook/assets/image%20%2864%29.png)

引入消息队列之后：

![](../.gitbook/assets/image%20%2863%29.png)

有了消息队列，从主动调用的方式，变成了消息的订阅发布，从而解耦。

### 1.2 异步消息

不过，使用消息队列进行异步处理，会有一个前提，返回的结果不依赖于处理的结果。

传统：

![](../.gitbook/assets/image%20%2862%29.png)

消息队列：

![](../.gitbook/assets/image%20%2868%29.png)

### 1.3 流量削锋

传统：

![](../.gitbook/assets/image%20%2869%29.png)

消息队列：

![](../.gitbook/assets/image%20%2867%29.png)

### 1.4 日志处理

![](../.gitbook/assets/image%20%2856%29.png)

```text
日志采集客户端，负责日志数据采集，定时批量写入 Kafka 队列。
Kafka 消息队列，负责日志数据的接收，存储和转发。
日志处理应用：订阅并消费 Kafka 队列中的日志数据。
```



## 2 消息队列的缺点：

### 2.1 系统可用性降低

系统引入外部依赖越多，越容易挂掉。引入了消息队列，就要考虑消息队列的可用性。

### 2.2 系统复杂度提高

需要多考虑：

```text
1、重复消费怎么办？
2、怎么保证消息不丢失
3、需要消息顺序的业务场景怎么办？
```

### 2.3 一致性问题

特别是使用消息队列用作异步处理，需要达到最终一致性。



## 3 消息队列的组成？

![](../.gitbook/assets/image%20%2858%29.png)

producer  
consumer  
broker： 消息代理，负责存储消息和转发消息。

## 4 消费语义

### 4.1 消息至多被消费一次

适合能容忍丢失消息的场景。

```text
1、Producer 发送消息到 Message Broker 阶段
Producer 发消息给Message Broker 时，不要求 Message Broker 对接收到的消息响应确认，Producer 也不用关心 Message Broker 是否收到消息了。
2、Message Broker 存储/转发阶段
对 Message Broker 的存储不要求持久性。
转发消息时，也不用关心 Consumer 是否真的收到了。
3、Consumer 消费阶段
Consumer 从 Message Broker 中获取到消息后，可以从 Message Broker 删除消息。
或 Message Broker 在消息被 Consumer 拿去消费时删除消息，不用关心 Consumer 最后对消息的消费情况如何。
```

### 4.2 消息至少被消费一次

适合不能容忍丢消息，允许重复消费的任务。

```text
1、Producer 发送消息到 Message Broker 阶段
Producer 发消息给 Message Broker ，Message Broker 必须响应对消息的确认。
2、Message Broker 存储/转发阶段
Message Broker 必须提供持久性保障。
转发消息时，Message Broker 需要 Consumer 通知删除消息，才能将消息删除。
3、Consumer消费阶段
Consumer 从 Message Broker 中获取到消息，必须在消费完成后，Message Broker上的消息才能被删除。
```

### 4.3 消息仅被消费一次

适合对消息消费情况要求非常高的任务，实现较为复杂。

#### 4.3.1 broker上存储的消息被consumer仅消费一次

1、producer --&gt; broker

producer 不关心broker是否收到消息，也不要求broker 收到消息之后确认。

2、broker 存储转发阶段

必须提供持久化保证。

每条消息都有唯一标识

3、broker--&gt; consumer 消费  
consumer  消费之后，必须记下标识，防止重复消费。



#### 4.3.2 producer 上产生的消息被consumer仅消费一次

1、producer --&gt; broker  
broker 必须响应生产者的消息，并且producer负责为该消息产生唯一标识，以防止consumer重复消费。

2、broker 存储转发阶段  
必须提供持久化保证。

每条消息都有唯一标识

3、broker--&gt; consumer 消费  
consumer  消费之后，必须记下标识，防止重复消费。





## 5 





## 疑问

1、kafka  在保证消息不丢失方面，是如何做的？



2、kafka 在保证消息不重复方面，是如何做的？



## 参考





## 




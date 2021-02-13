# ES 面试题

## 参考文章：

[**https://www.jianshu.com/p/642d5115724f**](https://www.jianshu.com/p/642d5115724f)\*\*\*\*

## 0 es 文件分布

## 

## 1 **详细描述一下Elasticsearch搜索的过程**

\*\*\*\*

## **2**  **详细描述一下Elasticsearch更新和删除文档的过程**

  
****

## **3 详细描述一下Elasticsearch索引文档的过程**

\*\*\*\*

\*\*\*\*

## **4   Elasticsearch是如何实现Master选举的**

## 

## 5  **在并发情况下，Elasticsearch如果保证读写一致？**

\*\*\*\*

\*\*\*\*

## 6 **Elasticsearch对于大数据量（上亿量级）的聚合如何实现？**

\*\*\*\*

## **7 对于GC方面，在使用Elasticsearch时要注意什么？**

## 

\*\*\*\*

## 8 **ES的heap被什么占用?**

\*\*\*\*

## 9 **那么有哪些途径减少data node上的segment memory占用呢？**

## **10** keyword类型 与 text 类型的区别

keyword 存储数据时候，不会分词建立索引；

text  存储数据时候，会自动分词，并生成索引。



在进行query 时， term 是不会分词的，也不会进行大小写的转换，精确匹配。如果对 字段进行了text 索引 \(加入插入了 "i am a boy", 此时进行了分词 “i”, "am" , "a", "boy"\)，那么使用term 查询相同的短语（比如“i am a boy"，因为没有分词 ），则查不出来。





## 11 

\*\*\*\*

\*\*\*\*


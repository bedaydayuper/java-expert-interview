# ES 面试题

## 参考文章：

[**https://www.jianshu.com/p/642d5115724f**](https://www.jianshu.com/p/642d5115724f)\*\*\*\*

## 0 es 文件分布



## 5  **在并发情况下，Elasticsearch如果保证读写一致？**

[https://www.jianshu.com/p/61dd9fb7d785](https://www.jianshu.com/p/61dd9fb7d785)    
通过版本号 使用乐观锁 保证数据的一致性。

另外对于写操作：

> 写操作一致性级别支持 quorum/one/all, 默认为quorum, 即只有当大多数分片可用时才允许写操作。

对于读操作:

> 可以设置replication 为sync （默认），这使得操作在主分片和副本分片都完成才会返回；如果设置为async 时，也可以通过设置搜索请求参数\_preference 为 primary 来查询主分片，确保文档是最新的。



## 6 **Elasticsearch对于大数据量（上亿量级）的聚合如何实现？**

\*\*\*\*

{% embed url="https://xiaozhazi.github.io/2020/08/10/ES-tech\_md/\#ES%E5%AF%B9%E4%BA%8E%E5%A4%A7%E6%95%B0%E6%8D%AE%E9%87%8F-%E4%B8%8A%E4%BA%BF%E9%87%8F%E7%BA%A7-%E7%9A%84%E8%81%9A%E5%90%88%E5%A6%82%E4%BD%95%E5%AE%9E%E7%8E%B0" %}

> ES目前支持两种近似算法\(cardinality和percentiles\),以牺牲一点小小的估算错误为代价换回高速的执行效率和极小的内存消耗.



\*\*\*\*

## **7 对于GC方面，在使用Elasticsearch时要注意什么？**

> 1）倒排词典的索引需要常驻内存，无法GC，需要监控data node上segment memory增长趋势。
>
>  2）各类缓存，field cache, filter cache, indexing cache, bulk queue等等，要设置合理的大小，并且要应该根据最坏的情况来看heap是否够用，也就是各类缓存全部占满的时候，还有heap空间可以分配给其他任务吗？避免采用clear cache等“自欺欺人”的方式来释放内存。 
>
> 3）避免返回大量结果集的搜索与聚合。确实需要大量拉取数据的场景，可以采用scan & scroll api来实现。 
>
> 4）cluster stats驻留内存并无法水平扩展，超大规模集群可以考虑分拆成多个集群通过tribe node连接。 
>
> 5）想知道heap够不够，必须结合实际应用场景，并对集群的heap使用情况做持续的监控



## **10** keyword类型 与 text 类型的区别

keyword 存储数据时候，不会分词建立索引；

text  存储数据时候，会自动分词，并生成索引。



在进行query 时， term 是不会分词的，也不会进行大小写的转换，精确匹配。如果对 字段进行了text 索引 \(加入插入了 "i am a boy", 此时进行了分词 “i”, "am" , "a", "boy"\)，那么使用term 查询相同的短语（比如“i am a boy"，因为没有分词 ），则查不出来。

全文本查询，则会进行分词。

## 11 query 跟filter 的区别

最佳实践：如果可能，使用filter 而不是 query.

<table>
  <thead>
    <tr>
      <th style="text-align:left">&#xA0; &#x6BD4;&#x8F83;&#x70B9;</th>
      <th style="text-align:left">query</th>
      <th style="text-align:left">filter</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">&#x5173;&#x6CE8;&#x70B9;</td>
      <td style="text-align:left">
        <p>&#x5339;&#x914D;&#x7A0B;&#x5EA6;&#x5982;&#x4F55;&#xFF1F;</p>
        <p>&#x5173;&#x6CE8;&#x5F97;&#x5206;</p>
        <p>&#x53EF;&#x4EE5;&#x505A;&#x5206;&#x8BCD;</p>
      </td>
      <td style="text-align:left">
        <p>&#x662F;&#x5426;&#x5339;&#x914D;</p>
        <p>&#x4E0D;&#x5173;&#x6CE8;&#x5F97;&#x5206;</p>
        <p>&#x9002;&#x7528;&#x4E8E;&#x5B8C;&#x5168;&#x7CBE;&#x786E;&#x5339;&#x914D;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">&#x662F;&#x5426;&#x7F13;&#x5B58;</td>
      <td style="text-align:left">
        <p>&#x5426;</p>
        <p>&#x56E0;&#x4E3A;&#x4E0D;&#x4EC5;&#x8981;&#x67E5;&#x627E;&#x5339;&#x914D;&#x7684;&#x6587;&#x6863;&#xFF0C;&#x8FD8;&#x5F97;&#x8BA1;&#x7B97;&#x6BCF;&#x4E2A;&#x6587;&#x6863;&#x7684;&#x76F8;&#x5173;&#x7A0B;&#x5EA6;&#x3002;</p>
      </td>
      <td style="text-align:left">
        <p>&#x662F;</p>
        <p>&#x7ECF;&#x5E38;&#x4F7F;&#x7528;&#x7684;&#x8FC7;&#x6EE4;&#x5668;&#x4F1A;&#x88AB;&#x81EA;&#x52A8;&#x7F13;&#x5B58;&#xFF0C;&#x63D0;&#x9AD8;&#x6027;&#x80FD;&#x3002;</p>
      </td>
    </tr>
  </tbody>
</table>

filter 的缓存机制：

> Elasticsearch将创建一个文档匹配过滤器的位集bitset（如果文档匹配则为1，否则为0）。 随后用相同的过滤器执行查询将重用此信息。
>
> 每当添加或更新新文档时，位集bitset也会更新。

## 12 es 的分页

[https://cloud.tencent.com/developer/article/1676915](https://cloud.tencent.com/developer/article/1676915) 

| 方案 | 优点 | 缺点 | 适用场景 |
| :--- | :--- | :--- | :--- |
| from size | 简单 | 耗内存 | 数据量少 |
| search after 方案 | 可以实时高效的进行分页查询 | 它只能做`下一页`这样的查询场景，不能随机的指定页数查询。 | 需要进行很深度的分页，但是可以不指定页数翻页，只要可以实时请求下一页就行 |
| scroll  | 高效 | 它基于快照，不能用在实时性高的业务场景 | 需要一次性或者每次查询大量的文档，但是对实时性要求并不高 |

## 

\*\*\*\*


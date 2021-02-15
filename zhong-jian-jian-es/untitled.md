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

## 12

\*\*\*\*

\*\*\*\*


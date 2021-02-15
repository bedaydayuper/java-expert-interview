# ES 日常操作汇总-Bucket聚合

> Bucket 可以理解为一个桶，会遍历文档中的内容，凡是符合某一要求的就放入一个桶，分桶相当于SQL 中的group by。

## 1 准备数据

```text
## 创建索引
PUT cars
{
  "mappings": {
    "cars_type": {
      "properties": {
        "price": {
          "type": "long"
        },
        "color": {
          "type": "keyword"
        },
        "brand": {
          "type": "keyword"
        },
        "sellTime": {
          "type": "date"
        }
      }
    }
  }
}
```

## 2 


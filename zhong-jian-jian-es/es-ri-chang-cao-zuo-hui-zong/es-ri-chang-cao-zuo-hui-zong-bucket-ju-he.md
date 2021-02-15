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

## 批量插入数据
POST /cars/cars_type/_bulk
{ "index": {}}
{ "price" : 80000, "color" : "red", "brand" : "BMW", "sellTime" : "2014-01-28" }
{ "index": {}}
{ "price" : 85000, "color" : "green", "brand" : "BMW", "sellTime" : "2014-02-05" }
{ "index": {}}
{ "price" : 120000, "color" : "green", "brand" : "Mercedes", "sellTime" : "2014-03-18" }
{ "index": {}}
{ "price" : 105000, "color" : "blue", "brand" : "Mercedes", "sellTime" : "2014-04-02" }
{ "index": {}}
{ "price" : 72000, "color" : "green", "brand" : "Audi", "sellTime" : "2014-05-19" }
{ "index": {}}
{ "price" : 60000, "color" : "red", "brand" : "Audi", "sellTime" : "2014-06-05" }
{ "index": {}}
{ "price" : 40000, "color" : "red", "brand" : "Audi", "sellTime" : "2014-07-01" }
{ "index": {}}
{ "price" : 35000, "color" : "blue", "brand" : "Honda", "sellTime" : "2014-08-12" }
```

## 2 Term Aggregation

> 根据某一项的每个唯一的值的聚合

```text
# 根据品牌分桶
GET cars/cars_type/_search
{
  "size": 0, 
  "aggs": {
    "group_by_brand": {
      "terms": {
        "field": "brand"
      }
    }
  }
  
结果：
{
  "took": 0,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "failed": 0
  },
  "hits": {
    "total": 8,
    "max_score": 0,
    "hits": []
  },
  "aggregations": {
    "group_by_brand": {
      "doc_count_error_upper_bound": 0,
      "sum_other_doc_count": 0,
      "buckets": [
        {
          "key": "Audi",
          "doc_count": 3
        },
        {
          "key": "BMW",
          "doc_count": 2
        },
        {
          "key": "Mercedes",
          "doc_count": 2
        },
        {
          "key": "Honda",
          "doc_count": 1
        }
      ]
    }
  }
}
```

分桶后，按照数量排序，然后显示前n.

```text
GET cars/cars_type/_search
{
  "size": 0, 
  "aggs": {
    "group_by_brand": {
      "terms": {
        "field": "brand",
        "order" : {"_count": "asc"},  // 按照数量 正序排列
        "size": 3 // 显示前3
      }
    }
  }
}
```

分桶后，显示数量大于3的桶

```text
GET cars/cars_type/_search
{
  "size": 0, 
  "aggs": {
    "group_by_brand": {
      "terms": {
        "field": "brand",
        "min_doc_count" : 2
      }
    }
  }
}
```

使用精确词条进行分桶

```text

GET cars/cars_type/_search
{
  "size": 0, 
    "aggs" : {
        "JapaneseCars" : {
             "terms" : {
                 "field" : "brand",
                 "include" : ["BMW"]
             }
         }
    }
}
```

## 3 Filter Aggregation

> 指具体的域 和具体的值，可以理解为在Terms Aggregation 的基础上进行了过滤，只对特定的值进行了聚合。

获取某个值的桶，然后求这个桶的平均值

```text
GET cars/cars_type/_search
{
  "size": 0,
  "aggs": {
    "brands": {
      "filter": {
        "term": {
          "brand": "BMW"
        }
      },
      "aggs": {
        "avg_price": {
          "avg": {
            "field": "price"
          }
        }
      }
    }
  }
}
```

Filters Aggregation 可以指定多个过滤条件 进行聚合。

```text
GET cars/cars_type/_search
{
  "size": 0,
  "aggs": {
    "cars": {
      "filters": {
        "filters": {
          "colorBucket": {
            "match": {
              "color": "red"
            }
          },
          "brandBucket": {
            "match": {
              "brand": "Audi"
            }
          }
        }
      }
    }
  }
}

结果：
{
  "took": 1,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "failed": 0
  },
  "hits": {
    "total": 8,
    "max_score": 0,
    "hits": []
  },
  "aggregations": {
    "cars": {
      "buckets": {
        "brandBucket": {
          "doc_count": 3
        },
        "colorBucket": {
          "doc_count": 3
        }
      }
    }
  }
}
```

## 4 Histogram Aggregation

> Histogram与Terms聚合类似，都是数据分组，区别是Terms是按照Field的值分组，而Histogram可以按照指定的间隔对Field进行分组.

```text

GET cars/cars_type/_search
{
  "size": 0,
  "aggs": {
    "prices": {
      "histogram": {
        "field": "price",
        "interval": 10000, // 按照10000 间隔，进行分桶
        "min_doc_count": 1 // 桶内至少为1个doc, 如果没有，则不显示
      }
    }
  }
}
```

## 5 Range Aggregation

> 根据用户传递的范围参数作为桶，进行相应的聚合。在同一个请求中，可以传递多组范围，每组范围作为一个桶。

```text
GET cars/cars_type/_search
{
  "size": 0,
  "aggs": {
    "price_ranges": {
      "range": {
        "field": "price",
        "ranges": [
          {
            "to": 50000
          },
          {
            "from": 5000,
            "to": 80000
          },
          {
            "from": 80000
          }
        ]
      }
    }
  }
}
```

## 6 Date Aggregation



按照每个月的销量进行分桶

> ```text
> GET cars/cars_type/_search
> {
>   "size": 0,
>   "aggs": {
>     "sales_over_time": {
>       "date_histogram": {
>         "field": "sellTime",
>         "interval": "1M",
>         "format": "yyyy-MM-dd"
>       }
>     }
>   }
> }
>
> 返回结果：
> {
>   "took": 5,
>   "timed_out": false,
>   "_shards": {
>     "total": 5,
>     "successful": 5,
>     "failed": 0
>   },
>   "hits": {
>     "total": 8,
>     "max_score": 0,
>     "hits": []
>   },
>   "aggregations": {
>     "sales_over_time": {
>       "buckets": [
>         {
>           "key_as_string": "2014-01-01",
>           "key": 1388534400000,
>           "doc_count": 1
>         },
>         {
>           "key_as_string": "2014-02-01",
>           "key": 1391212800000,
>           "doc_count": 1
>         },
>         {
>           "key_as_string": "2014-03-01",
>           "key": 1393632000000,
>           "doc_count": 1
>         },
>         {
>           "key_as_string": "2014-04-01",
>           "key": 1396310400000,
>           "doc_count": 1
>         },
>         {
>           "key_as_string": "2014-05-01",
>           "key": 1398902400000,
>           "doc_count": 1
>         },
>         {
>           "key_as_string": "2014-06-01",
>           "key": 1401580800000,
>           "doc_count": 1
>         },
>         {
>           "key_as_string": "2014-07-01",
>           "key": 1404172800000,
>           "doc_count": 1
>         },
>         {
>           "key_as_string": "2014-08-01",
>           "key": 1406851200000,
>           "doc_count": 1
>         }
>       ]
>     }
>   }
> }
> ```

按照指定时间区间分桶

```text
# 10个月前的分为一个桶，10个月前之后的分为一个桶
GET cars/cars_type/_search
{
  "size": 0,
  "aggs": {
    "range": {
      "date_range": {
        "field": "sellTime",
        "format": "MM-yyyy",
        "ranges": [
          {
            "to": "now-10M/M"
          },
          {
            "from": "now-10M/M"
          }
        ]
      }
    }
  }
}

返回结果：

{
  "took": 4,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "failed": 0
  },
  "hits": {
    "total": 8,
    "max_score": 0,
    "hits": []
  },
  "aggregations": {
    "range": {
      "buckets": [
        {
          "key": "*-04-2020",
          "to": 1585699200000,
          "to_as_string": "04-2020",
          "doc_count": 8
        },
        {
          "key": "04-2020-*",
          "from": 1585699200000,
          "from_as_string": "04-2020",
          "doc_count": 0
        }
      ]
    }
  }
}
```


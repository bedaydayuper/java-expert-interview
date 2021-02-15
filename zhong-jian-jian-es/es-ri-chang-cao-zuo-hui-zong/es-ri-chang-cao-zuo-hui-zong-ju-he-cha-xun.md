# ES 日常操作汇总-聚合查询

## 0 引入

聚合分析主要有如下四类：

> Metric\(指标\): 指标分析类型，如计算最大值、最小值、平均值等等 （对桶内的文档进行聚合分析的操作） 
>
> Bucket\(桶\): 分桶类型，类似SQL中的GROUP BY语法 （满足特定条件的文档的集合） 
>
> Pipeline\(管道\): 管道分析类型，基于上一级的聚合分析结果进行在分析 
>
> Matrix\(矩阵\): 矩阵分析类型（聚合是一种面向数值型的聚合，用于计算一组文档字段中的统计信息）

常见的写法：

```text
"aggregations" : {
    "<aggregation_name>" : {                                 <!--聚合的名字 -->
        "<aggregation_type>" : {                               <!--聚合的类型 -->
            <aggregation_body>                                 <!--聚合体：对哪些字段进行聚合 -->
        }
        [,"meta" : {  [<meta_data_body>] } ]?               <!--元 -->
        [,"aggregations" : { [<sub_aggregation>]+ } ]?   <!--在聚合里面在定义子聚合 -->
    }
    [,"<aggregation_name_2>" : { ... } ]*                     <!--聚合的名字 -->
}
```

## 1 metric 聚合

> metric 聚合分析分为 单值分析和多值分析 两类。
>
> 单值：
>
> min, max, avg, sum, cardinality\(不重复字段个数\)
>
> 多值：  
> stats, extended\_\__stats, percentile, percentile\_rank , top hits_

### 1.1 准备数据

```text
## 创建索引mapping
PUT /employees/
{
  "mappings" : {
    "employees_type": {
      "properties" : {
        "age" : {
          "type" : "integer"
        },
        "gender" : {
          "type" : "keyword"
        },
        "job" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 50
            }
          }
        },
        "name" : {
          "type" : "keyword"
        },
        "salary" : {
          "type" : "integer"
        }
      }
    }
    }
      
}

## 批量插入数据
PUT /employees/employees_type/_bulk
{"index":{"_id":"1"}}
{"name":"Emma","age":32,"job":"Product Manager","gender":"female","salary":35000}
{"index":{"_id":"2"}}
{"name":"Underwood","age":41,"job":"Dev Manager","gender":"male","salary":50000}
{"index":{"_id":"3"}}
{"name":"Tran","age":25,"job":"Web Designer","gender":"male","salary":18000}
{"index":{"_id":"4"}}
{"name":"Rivera","age":26,"job":"Web Designer","gender":"female","salary":22000}
{"index":{"_id":"5"}}
{"name":"Rose","age":25,"job":"QA","gender":"female","salary":18000}
{"index":{"_id":"6"}}
{"name":"Lucy","age":31,"job":"QA","gender":"female","salary":25000}
{"index":{"_id":"7"}}
{"name":"Byrd","age":27,"job":"QA","gender":"male","salary":20000}
{"index":{"_id":"8"}}
{"name":"Foster","age":27,"job":"Java Programmer","gender":"male","salary":20000}
{"index":{"_id":"9"}}
{"name":"Gregory","age":32,"job":"Java Programmer","gender":"male","salary":22000}
{"index":{"_id":"10"}}
{"name":"Bryant","age":20,"job":"Java Programmer","gender":"male","salary":9000}


```

### 1.2  最小值 / 最大值/ 平均值/ 总和 

```text
# 最小值
POST employees/employees_type/_search
{
  "size": 0,  // size 是显示命中检索条件的doc 数量，跟 求最小值没有关系。
  "aggs": {
    "min_salary": {
      "min": {       // 这儿是agg 的类型
        "field": "salary"
      }
    }
  }
}
# 最大值
POST employees/employees_type/_search
{
  "size": 0, 
  "aggs": {
    "max_salary": {
      "max": {
        "field": "salary"
      }
    }
  }
}

# 平均值
POST employees/employees_type/_search
{
  "size": 0, 
  "aggs": {
    "avg_salary": {
      "avg": {
        "field": "salary"
      }
    }
  }
}

# 求和， 基于查询条件的求和。此时的结果是满足查询条件的求和。
GET employees/employees_type/_search
{
  "query": {
    "range": {
      "salary": {
        "gte": 35000
      }
    }
  },
  "_source": "salary",
  "aggs": {
    "sum_salary": {
      "sum": {
        "field": "salary"
      }
    }
  }
}

## 唯一值   下面是求 salary 有多少种，类似于mysql 的 distinct 

POST employees/employees_type/_search
{
  "query": {
    "range": {
      "salary": {
        "gte": 10
      }
    }
  }, 
  "_source": "salary", 
  "sort": [
    {
      "salary": {
        "order": "desc"
      }
    }
  ], 
  "aggs": {
    "salary_count": {
      "cardinality": {
        "field": "salary"
      }
    }
  }
}

```

### 1.3 stats 统计，请求后会直接显示多种聚合结果

```text
POST employees/employees_type/_search
{
  "query": {
    "range": {
      "salary": {
        "gte": 9001
      }
    }
  }, 
  "_source": "salary", 
  "sort": [
    {
      "salary": {
        "order": "desc"
      }
    }
  ], 
  "aggs": {
    "salary_stats": {
      "stats": {
        "field": "salary"
      }
    }
  }
}
返回：
{
  "took": 1,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "failed": 0
  },
  "hits": {
    "total": 9,
    "max_score": null,
    "hits": [
      {
        "_index": "employees",
        "_type": "employees_type",
        "_id": "2",
        "_score": null,
        "_source": {
          "salary": 50000
        },
        "sort": [
          50000
        ]
      },
      {
        "_index": "employees",
        "_type": "employees_type",
        "_id": "1",
        "_score": null,
        "_source": {
          "salary": 35000
        },
        "sort": [
          35000
        ]
      },
      {
        "_index": "employees",
        "_type": "employees_type",
        "_id": "6",
        "_score": null,
        "_source": {
          "salary": 25000
        },
        "sort": [
          25000
        ]
      },
      {
        "_index": "employees",
        "_type": "employees_type",
        "_id": "9",
        "_score": null,
        "_source": {
          "salary": 22000
        },
        "sort": [
          22000
        ]
      },
      {
        "_index": "employees",
        "_type": "employees_type",
        "_id": "4",
        "_score": null,
        "_source": {
          "salary": 22000
        },
        "sort": [
          22000
        ]
      },
      {
        "_index": "employees",
        "_type": "employees_type",
        "_id": "8",
        "_score": null,
        "_source": {
          "salary": 20000
        },
        "sort": [
          20000
        ]
      },
      {
        "_index": "employees",
        "_type": "employees_type",
        "_id": "7",
        "_score": null,
        "_source": {
          "salary": 20000
        },
        "sort": [
          20000
        ]
      },
      {
        "_index": "employees",
        "_type": "employees_type",
        "_id": "5",
        "_score": null,
        "_source": {
          "salary": 18000
        },
        "sort": [
          18000
        ]
      },
      {
        "_index": "employees",
        "_type": "employees_type",
        "_id": "3",
        "_score": null,
        "_source": {
          "salary": 18000
        },
        "sort": [
          18000
        ]
      }
    ]
  },
  "aggregations": {
    "salary_stats": {
      "count": 9,
      "min": 18000,
      "max": 50000,
      "avg": 25555.555555555555,
      "sum": 230000
    }
  }
}
```

### 1.4 percentiles

> 对指定字段的值按从小到大累计每个值对应的文档数的占比，返回指定占比比例对应的值。



**默认按照\[ 1, 5, 25, 50, 75, 95, 99 \]来统计**

```text
POST employees/employees_type/_search?filter_path=hits.hits._source,hits.total,aggregations
{
  "query": {
    "range": {
      "salary": {
        "gte": 9001
      }
    }
  }, 
  "_source": "salary", 
  "sort": [
    {
      "salary": {
        "order": "desc"
      }
    }
  ], 
  "aggs": {
    "age_percentiles": {
      "percentiles": {
        "field": "age"
      }
    }
  }
}

返回结果：
{
  "hits": {
    "total": 9,
    "hits": [
      {
        "_source": {
          "salary": 50000
        }
      },
      {
        "_source": {
          "salary": 35000
        }
      },
      {
        "_source": {
          "salary": 25000
        }
      },
      {
        "_source": {
          "salary": 22000
        }
      },
      {
        "_source": {
          "salary": 22000
        }
      },
      {
        "_source": {
          "salary": 20000
        }
      },
      {
        "_source": {
          "salary": 20000
        }
      },
      {
        "_source": {
          "salary": 18000
        }
      },
      {
        "_source": {
          "salary": 18000
        }
      }
    ]
  },
  "aggregations": {
    "age_percentiles": {
      "values": {
        "1.0": 25,
        "5.0": 25,
        "25.0": 26,
        "50.0": 27,
        "75.0": 32,
        "95.0": 37.4,
        "99.0": 40.28
      }
    }
  }
}
```

也可以指定分位值

```text
POST employees/employees_type/_search?filter_path=hits.hits._source,hits.total,aggregations
{
  "query": {
    "range": {
      "salary": {
        "gte": 9001
      }
    }
  }, 
  "_source": "salary", 
  "sort": [
    {
      "salary": {
        "order": "desc"
      }
    }
  ], 
  "aggs": {
    "age_percentiles": {
      "percentiles": {
        "field": "age",
        "percents": [
          95,
          96,
          99
        ]
      }
    }
  }
}

```



### 1.5 Percentile Ranks 通过文档值 求百分比

```text
POST employees/employees_type/_search?filter_path=hits.hits._source,hits.total,aggregations
{
  "query": {
    "range": {
      "salary": {
        "gte": 9001
      }
    }
  }, 
  "_source": "age", 
  "sort": [
    {
      "salary": {
        "order": "desc"
      }
    }
  ], 
  "aggs": {
    "age_percentiles_ranks": {
      "percentile_ranks": {
        "field": "age",
        "values": [
          30, 25
        ]
      }
    }
  }
}

返回结果

{
  "hits": {
    "total": 9,
    "hits": [
      {
        "_source": {
          "age": 41
        }
      },
      {
        "_source": {
          "age": 32
        }
      },
      {
        "_source": {
          "age": 31
        }
      },
      {
        "_source": {
          "age": 32
        }
      },
      {
        "_source": {
          "age": 26
        }
      },
      {
        "_source": {
          "age": 27
        }
      },
      {
        "_source": {
          "age": 27
        }
      },
      {
        "_source": {
          "age": 25
        }
      },
      {
        "_source": {
          "age": 25
        }
      }
    ]
  },
  "aggregations": {
    "age_percentiles_ranks": {
      "values": {
        "25.0": 11.11111111111111,
        "30.0": 60.00000000000001
      }
    }
  }
}
```

### 1.6 Top Hits 分桶后获取该桶内前n 的文档列表

```text
## 根据工作类型分桶，然后按照性别分桶，计算每个桶中工资的top2的薪资。

POST employees/employees_type/_search?filter_path=hits.hits._source,hits.total,aggregations
{
  "size": 0,
  "aggs": {
    "Job_gender_stats": {
      "terms": {
        "field": "job.keyword"
      },
      "aggs": {
        "gender_stats": {
          "terms": {
            "field": "gender"
          },
          "aggs": {
            "salary_top_hits": {
              "top_hits": {
                "sort": [{
                  "salary": {"order": "desc"}
                }
                ],
                "size": 2
              }
            }
          }
        }
      }
    }
  }
}

返回结果：
{
  "hits": {
    "total": 10
  },
  "aggregations": {
    "Job_gender_stats": {
      "doc_count_error_upper_bound": 0,
      "sum_other_doc_count": 0,
      "buckets": [
        {
          "key": "Java Programmer",
          "doc_count": 3,
          "gender_stats": {
            "doc_count_error_upper_bound": 0,
            "sum_other_doc_count": 0,
            "buckets": [
              {
                "key": "male",
                "doc_count": 3,
                "salary_top_hits": {
                  "hits": {
                    "total": 3,
                    "max_score": null,
                    "hits": [
                      {
                        "_index": "employees",
                        "_type": "employees_type",
                        "_id": "9",
                        "_score": null,
                        "_source": {
                          "name": "Gregory",
                          "age": 32,
                          "job": "Java Programmer",
                          "gender": "male",
                          "salary": 22000
                        },
                        "sort": [
                          22000
                        ]
                      },
                      {
                        "_index": "employees",
                        "_type": "employees_type",
                        "_id": "8",
                        "_score": null,
                        "_source": {
                          "name": "Foster",
                          "age": 27,
                          "job": "Java Programmer",
                          "gender": "male",
                          "salary": 20000
                        },
                        "sort": [
                          20000
                        ]
                      }
                    ]
                  }
                }
              }
            ]
          }
        },
        {
          "key": "QA",
          "doc_count": 3,
          "gender_stats": {
            "doc_count_error_upper_bound": 0,
            "sum_other_doc_count": 0,
            "buckets": [
              {
                "key": "female",
                "doc_count": 2,
                "salary_top_hits": {
                  "hits": {
                    "total": 2,
                    "max_score": null,
                    "hits": [
                      {
                        "_index": "employees",
                        "_type": "employees_type",
                        "_id": "6",
                        "_score": null,
                        "_source": {
                          "name": "Lucy",
                          "age": 31,
                          "job": "QA",
                          "gender": "female",
                          "salary": 25000
                        },
                        "sort": [
                          25000
                        ]
                      },
                      {
                        "_index": "employees",
                        "_type": "employees_type",
                        "_id": "5",
                        "_score": null,
                        "_source": {
                          "name": "Rose",
                          "age": 25,
                          "job": "QA",
                          "gender": "female",
                          "salary": 18000
                        },
                        "sort": [
                          18000
                        ]
                      }
                    ]
                  }
                }
              },
              {
                "key": "male",
                "doc_count": 1,
                "salary_top_hits": {
                  "hits": {
                    "total": 1,
                    "max_score": null,
                    "hits": [
                      {
                        "_index": "employees",
                        "_type": "employees_type",
                        "_id": "7",
                        "_score": null,
                        "_source": {
                          "name": "Byrd",
                          "age": 27,
                          "job": "QA",
                          "gender": "male",
                          "salary": 20000
                        },
                        "sort": [
                          20000
                        ]
                      }
                    ]
                  }
                }
              }
            ]
          }
        },
        {
          "key": "Web Designer",
          "doc_count": 2,
          "gender_stats": {
            "doc_count_error_upper_bound": 0,
            "sum_other_doc_count": 0,
            "buckets": [
              {
                "key": "female",
                "doc_count": 1,
                "salary_top_hits": {
                  "hits": {
                    "total": 1,
                    "max_score": null,
                    "hits": [
                      {
                        "_index": "employees",
                        "_type": "employees_type",
                        "_id": "4",
                        "_score": null,
                        "_source": {
                          "name": "Rivera",
                          "age": 26,
                          "job": "Web Designer",
                          "gender": "female",
                          "salary": 22000
                        },
                        "sort": [
                          22000
                        ]
                      }
                    ]
                  }
                }
              },
              {
                "key": "male",
                "doc_count": 1,
                "salary_top_hits": {
                  "hits": {
                    "total": 1,
                    "max_score": null,
                    "hits": [
                      {
                        "_index": "employees",
                        "_type": "employees_type",
                        "_id": "3",
                        "_score": null,
                        "_source": {
                          "name": "Tran",
                          "age": 25,
                          "job": "Web Designer",
                          "gender": "male",
                          "salary": 18000
                        },
                        "sort": [
                          18000
                        ]
                      }
                    ]
                  }
                }
              }
            ]
          }
        },
        {
          "key": "Dev Manager",
          "doc_count": 1,
          "gender_stats": {
            "doc_count_error_upper_bound": 0,
            "sum_other_doc_count": 0,
            "buckets": [
              {
                "key": "male",
                "doc_count": 1,
                "salary_top_hits": {
                  "hits": {
                    "total": 1,
                    "max_score": null,
                    "hits": [
                      {
                        "_index": "employees",
                        "_type": "employees_type",
                        "_id": "2",
                        "_score": null,
                        "_source": {
                          "name": "Underwood",
                          "age": 41,
                          "job": "Dev Manager",
                          "gender": "male",
                          "salary": 50000
                        },
                        "sort": [
                          50000
                        ]
                      }
                    ]
                  }
                }
              }
            ]
          }
        },
        {
          "key": "Product Manager",
          "doc_count": 1,
          "gender_stats": {
            "doc_count_error_upper_bound": 0,
            "sum_other_doc_count": 0,
            "buckets": [
              {
                "key": "female",
                "doc_count": 1,
                "salary_top_hits": {
                  "hits": {
                    "total": 1,
                    "max_score": null,
                    "hits": [
                      {
                        "_index": "employees",
                        "_type": "employees_type",
                        "_id": "1",
                        "_score": null,
                        "_source": {
                          "name": "Emma",
                          "age": 32,
                          "job": "Product Manager",
                          "gender": "female",
                          "salary": 35000
                        },
                        "sort": [
                          35000
                        ]
                      }
                    ]
                  }
                }
              }
            ]
          }
        }
      ]
    }
  }
}

```

![](../../.gitbook/assets/image%20%2810%29.png)



## 2   Bucket 聚合


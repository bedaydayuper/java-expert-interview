# ES 日常操作汇总-复合查询

## 1 bool query

> bool 查询包含四种操作符，分别是如下四种：
>
> must: 必须匹配，贡献算分
>
> must\_not  过滤子句，必须不能匹配，但不贡献算分
>
> should: 选择性匹配，至少满足一条。贡献算分
>
> filter：过滤子句，必须匹配，但不贡献算分

官方 demo:

```text

POST _search
{
  "query": {
    "bool" : {
      "must" : {
        "term" : { "user" : "kimchy" }
      },
      "filter": {
        "term" : { "tag" : "tech" }
      },
      "must_not" : {
        "range" : {
          "age" : { "gte" : 10, "lte" : 20 }
        }
      },
      "should" : [
        { "term" : { "tag" : "wow" } },
        { "term" : { "tag" : "elasticsearch" } }
      ],
      "minimum_should_match" : 1,
      "boost" : 1.0
    }
  }
}
```



实战 demo:  \(下面例子中既有must 又有should, 则 以must 为主 查出数据，而should 贡献了算分\)

没有should 的demo:

```text
POST student/_search
{
  "query": {
    "bool" : {
      "must" : {
        "term" : { "interests" : "演" }
      },
      "must_not": {
        "range" : {
          "age" : {"gte" : 60}
        } 
      }
    }
  }
}
结果：
{
  "took": 0,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "failed": 0
  },
  "hits": {
    "total": 2,
    "max_score": 0.57843524,
    "hits": [
      {
        "_index": "student",
        "_type": "student_type",
        "_id": "2",
        "_score": 0.57843524,
        "_source": {
          "name": "刘德华",
          "address": "香港",
          "age": 28,
          "interests": "演戏 旅游",
          "birthday": "1980-06-19"
        }
      },
      {
        "_index": "student",
        "_type": "student_type",
        "_id": "5",
        "_score": 0.57843524,
        "_source": {
          "name": "向华强",
          "address": "香港",
          "age": 31,
          "interests": "演戏 主持",
          "birthday": "1958-06-19"
        }
      }
    ]
  }
}
```

增加should: 

```text
POST student/_search
{
  "query": {
    "bool" : {
      "must" : {
        "term" : { "interests" : "演" }
      },
      "must_not": {
        "range" : {
          "age" : {"gte" : 60}
        } 
      },
      "should": {
        "term" : {"name" : "强"}  
      }
    }
  }
}

结果：
{
  "took": 4,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "failed": 0
  },
  "hits": {
    "total": 2,
    "max_score": 1.7983742,
    "hits": [
      {
        "_index": "student",
        "_type": "student_type",
        "_id": "5",
        "_score": 1.7983742,
        "_source": {
          "name": "向华强",
          "address": "香港",
          "age": 31,
          "interests": "演戏 主持",
          "birthday": "1958-06-19"
        }
      },
      {
        "_index": "student",
        "_type": "student_type",
        "_id": "2",
        "_score": 0.57843524,
        "_source": {
          "name": "刘德华",
          "address": "香港",
          "age": 28,
          "interests": "演戏 旅游",
          "birthday": "1980-06-19"
        }
      }
    ]
  }
}

```

增加 filter:

```text
POST student/_search
{
  "query": {
    "bool" : {
      "must" : {
        "term" : { "interests" : "演" }
      },
      "must_not": {
        "range" : {
          "age" : {"gte" : 60}
        } 
      },
      "should": {
        "term" : {"name" : "强"}
      },
      "filter": {
          "range": {
            "birthday" : {
              "gte": "1960-06-19"
            }
          }
      }
    }
  }
}
返回结果：
{
  "took": 4,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "failed": 0
  },
  "hits": {
    "total": 1,
    "max_score": 0.57843524,
    "hits": [
      {
        "_index": "student",
        "_type": "student_type",
        "_id": "2",
        "_score": 0.57843524,
        "_source": {
          "name": "刘德华",
          "address": "香港",
          "age": 28,
          "interests": "演戏 旅游",
          "birthday": "1980-06-19"
        }
      }
    ]
  }
}
```

## 2 关于bool query 中should 的用法

> 所有 `must` 语句必须匹配，所有 `must_not` 语句都必须不匹配，但有多少 `should` 语句应该匹配呢？默认情况下，没有 `should` 语句是必须匹配的，只有一个例外：那就是当没有 `must` 语句的时候，至少有一个 `should` 语句必须匹配。

[https://www.elastic.co/guide/cn/elasticsearch/guide/current/bool-query.html](https://www.elastic.co/guide/cn/elasticsearch/guide/current/bool-query.html)  


```text
## 此时，有must, 所以should 可以匹配，也可以不匹配。 
## 所以刘德华 这条数据就出来了。
POST student/_search
{
  "query": {
    "bool" : {
      "must" : {
        "term" : { "interests" : "演" }
      },
      "must_not": {
        "range" : {
          "age" : {"gte" : 60}
        } 
      },
      "should": {
        "term" : {"name" : "强"} 
      }
    }
  }
}
结果：
{
  "took": 0,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "failed": 0
  },
  "hits": {
    "total": 2,
    "max_score": 1.7983742,
    "hits": [
      {
        "_index": "student",
        "_type": "student_type",
        "_id": "5",
        "_score": 1.7983742,
        "_source": {
          "name": "向华强",
          "address": "香港",
          "age": 31,
          "interests": "演戏 主持",
          "birthday": "1958-06-19"
        }
      },
      {
        "_index": "student",
        "_type": "student_type",
        "_id": "2",
        "_score": 0.57843524,
        "_source": {
          "name": "刘德华",
          "address": "香港",
          "age": 28,
          "interests": "演戏 旅游",
          "birthday": "1980-06-19"
        }
      }
    ]
  }
}
```

再来一个只有should 的demo

```text
## 因为只有should, 所以刘德华 这条数据就不符合要求了。
POST student/_search
{
  "query": {
    "bool" : {
      "should": {
        "term" : {"name" : "强"}  
      }
    }
  }
}
结果：
{
  "took": 0,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "failed": 0
  },
  "hits": {
    "total": 1,
    "max_score": 1.219939,
    "hits": [
      {
        "_index": "student",
        "_type": "student_type",
        "_id": "5",
        "_score": 1.219939,
        "_source": {
          "name": "向华强",
          "address": "香港",
          "age": 31,
          "interests": "演戏 主持",
          "birthday": "1958-06-19"
        }
      }
    ]
  }
}
```

没有must, 有must\_not, 有 should 会怎样呢？  
&gt; 此时，should 也必须满足一个条件。

看如下的例子：

```text
## 只有must_not 时查询：

POST student/_search
{
  "query": {
    "bool" : {
      "must_not": {
        "range" : {
          "age" : {"gte" : 60}
        } 
      }
    }
  }
}
结果 有多条：
{
  "took": 0,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "failed": 0
  },
  "hits": {
    "total": 4,
    "max_score": 1,
    "hits": [
      {
        "_index": "student",
        "_type": "student_type",
        "_id": "1",
        "_score": 1,
        "_source": {
          "name": "徐小小",
          "address": "杭州",
          "age": 3,
          "interests": "唱歌 画画  跳舞",
          "birthday": "2017-06-19"
        }
      },
      {
        "_index": "student",
        "_type": "student_type",
        "_id": "2",
        "_score": 1,
        "_source": {
          "name": "刘德华",
          "address": "香港",
          "age": 28,
          "interests": "演戏 旅游",
          "birthday": "1980-06-19"
        }
      },
      {
        "_index": "student",
        "_type": "student_type",
        "_id": "3",
        "_score": 1,
        "_source": {
          "name": "张小斐",
          "address": "北京",
          "age": 28,
          "interests": "小品 旅游",
          "birthday": "1990-06-19"
        }
      },
      {
        "_index": "student",
        "_type": "student_type",
        "_id": "5",
        "_score": 1,
        "_source": {
          "name": "向华强",
          "address": "香港",
          "age": 31,
          "interests": "演戏 主持",
          "birthday": "1958-06-19"
        }
      }
    ]
  }
}


## 如果有must_not  跟 should
POST student/_search
{
  "query": {
    "bool" : {
      "must_not": {
        "range" : {
          "age" : {"gte" : 60}
        } 
      },
      "should": {
        "term" : {"name" : "强"}  
      }
    }
  }
}

结果：（也只有一条）
{
  "took": 0,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "failed": 0
  },
  "hits": {
    "total": 1,
    "max_score": 1.219939,
    "hits": [
      {
        "_index": "student",
        "_type": "student_type",
        "_id": "5",
        "_score": 1.219939,
        "_source": {
          "name": "向华强",
          "address": "香港",
          "age": 31,
          "interests": "演戏 主持",
          "birthday": "1958-06-19"
        }
      }
    ]
  }
}
```



## 3 boosting query  不常用

must\_not 直接将doc 排除掉，有时需要将包含某些字符的doc 权重降低，此时可以使用 boosting query。  


> boosting 需要搭配 positive, negative, negative\_boost 。 只有匹配了 **positive查询** 的文档才会被包含到结果集中，但是同时匹配了**negative查询** 的文档会被降低其相关度，

```text
# 创建索引并添加数据
POST /news/news_type/_bulk
{ "index": { "_id": 1 }}
{ "content":"Apple Mac" }
{ "index": { "_id": 2 }}
{ "content":"Apple iPad" }
{ "index": { "_id": 3 }}
{ "content":"Apple employee like Apple Pie and Apple Juice" }



# 通过Boosting的方式，将3的记录也纳入结果集，只是排名会靠后。(结果 1->2->3)
POST news/news_type/_search
{
  "query": {
    "boosting": {
      "positive": {
        "match": {
          "content": "apple"
        }
      },
      "negative": {
        "match": {
          "content": "pie"
        }
      },
      "negative_boost": 0.5
    }
  }
}
```

## 4 constant\_score （固定分数） 不常用

暂时忽略，不常用。可以参考：

[https://www.elastic.co/guide/cn/elasticsearch/guide/current/ignoring-tfidf.html](https://www.elastic.co/guide/cn/elasticsearch/guide/current/ignoring-tfidf.html)

## 5 dis\_max （最佳匹配查询） 不常用

暂时忽略，不常用。

## 6 function\_score 函数查询 不常用

暂时忽略，不常用。

## 参考文献：

[https://www.cnblogs.com/qdhxhz/p/11529107.html](https://www.cnblogs.com/qdhxhz/p/11529107.html) 


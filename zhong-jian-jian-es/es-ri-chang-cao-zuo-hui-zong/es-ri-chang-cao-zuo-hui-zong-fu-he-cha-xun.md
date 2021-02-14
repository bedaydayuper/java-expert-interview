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

## 2 boosting query 



## 3 constant\_score （固定分数）

## 4 dis\_max （最佳匹配查询）

## 5 function\_score 函数查询

## 参考文献：

[https://www.cnblogs.com/qdhxhz/p/11529107.html](https://www.cnblogs.com/qdhxhz/p/11529107.html) 


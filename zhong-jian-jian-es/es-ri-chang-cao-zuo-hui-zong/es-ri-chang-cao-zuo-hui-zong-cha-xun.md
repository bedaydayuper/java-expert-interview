# ES 日常操作汇总-Query查询和Filter查询

> Query context： 查询上下文，既要计算文档是否匹配，又要算分，分数越高，匹配度越高。
>
> Filter context: 过滤上下文，只关心是否匹配，不关心算法。

## 0 数据准备

```text
# 创建
DELETE student

PUT student
{
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 1
  },
  "mappings": {
    "student_type": {
      "properties": {
        "name": {
          "type": "text"
        },
        "address": {
          "type": "keyword"
        },
        "age": {
          "type": "integer"
        },
        "interests": {
          "type": "text"
        },
        "birthday": {
          "type": "date"
        }
      }
    }
  }
}

POST /student/student_type/1
{
  "name":"徐小小",
  "address":"杭州",
  "age":3,
  "interests":"唱歌 画画  跳舞",
  "birthday":"2017-06-19"
}
POST /student/student_type/1
{
  "name":"徐小小",
  "address":"杭州",
  "age":3,
  "interests":"唱歌 画画  跳舞",
  "birthday":"2017-06-19"
}

POST /student/student_type/2
{
  "name":"刘德华",
  "address":"香港",
  "age":28,
  "interests":"演戏 旅游",
  "birthday":"1980-06-19"
}

POST /student/student_type/3
{
  "name":"张小斐",
  "address":"北京",
  "age":28,
  "interests":"小品 旅游",
  "birthday":"1990-06-19"
}

POST /student/student_type/4
{
  "name":"王小宝",
  "address":"德州",
  "age":63,
  "interests":"演戏 小品 打牌",
  "birthday":"1956-06-19"
}

POST /student/student_type/5
{
  "name":"向华强",
  "address":"香港",
  "age":31,
  "interests":"演戏 主持",
  "birthday":"1958-06-19"
}
```

## 1 Query 查询

### 1.1 match 查询

> match query: 
>
> match\_all 查询所有
>
> multi\_match: 查多个字段
>
> match\_phrase 短语查询。 首先分析查询字符串，从分析后的文本中构建短语查询，这意味着必须匹配短  
> 语中的所有分词。

数字完全匹配

```text
GET student/student_type/_search
{
  "query": {
    "match": {
      "age": 3   // age 为字段，3 为字段值
    }
  }
}
```

type 为text 的分词匹配

```text
GET student/student_type/_search
{
  "query": {
    "match": {
      "interests": "演戏"   // interests 的类型为type, 此时只要interests包含'演戏','演','戏'的都会命中
    }
  }
}
```

查询所有  


```text
GET student/student_type/_search
{
  "query": {
    "match_all": {
    
    }
  }
}
```

查询多个字段中，都包含某个值的情况

```text
GET student/student_type/_search
{
  "query": {
    "multi_match": {
      "query": "",   // 具体的值
      "fields": [] // 涉及的字段
    }
  }
}
```

match\_phrase 查询，需要完全匹配

```text
GET student/_search
{
  "query":{
    "match_phrase":{"interests": "演员"}   // 需要真正包含。一条都不会返回。
  }
}

如果改为match，则只要匹配任何一个字"演员", "演"， “员” 都会返回.

GET student/_search
{
  "query":{
    "match":{"interests": "演员"}   // 会返回3个。
  }
}
```

keyword 类型的字段，需要精确匹配

```text
GET student/_search
{
  "query":{
    "match":{"address": "广州"}   // address 是keyword 类型，所以需要精确匹配，这儿一条都不会返回。即使命中了“州” 字也不行。
  }
}
```

### 

### 1.2 term 查询和terms 查询



## 2 Filter 查询 

## 参考文献：

[https://www.cnblogs.com/qdhxhz/p/11493677.html](https://www.cnblogs.com/qdhxhz/p/11493677.html)


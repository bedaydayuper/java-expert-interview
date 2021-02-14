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



## 2 Filter 查询 


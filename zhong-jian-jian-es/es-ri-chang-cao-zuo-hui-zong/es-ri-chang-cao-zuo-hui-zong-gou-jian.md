# ES 日常操作汇总-文档与索引的CRUD

## 1 索引CRUD

### 1.1 查询

```text
GET /_cat/indices?v&pretty

结果举例：

health status index      uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   bank       8SE_rI-AQc2wKiDF4DW1lA   5   1       1000            0    640.6kb        640.6kb

结果分析：
store.size 磁盘占用大小
pri.store.size 主分片磁盘占用大小

```

### 1.2 创建

```text
PUT /student
{
  "settings": {    //  索引的settings.
    "number_of_shards": 3, // 每个索引的主分片数，默认值是 5 . 这个配置在索引创建后不能修改。
    "number_of_replicas": 1 // 每个主分片的副本数，默认值是 1 。对于活动的索引库，这个配置可以随时修改。
  },
  "mappings": {   // 索引的mappings
    "demoType": {    // type, 在新的版本里面 type 不再使用。
      "properties": {   // 属性
        "name": {
          "type": "text", // 类型
          "index": analyzed, // 索引类型
          "analyzer": "english", // 分词类型， 插入文档时，将text类型的字段做分词然后插入倒排索引
          "search_analyzer": "english"  // 搜索时的分词类型，在查询时，先对要查询的text类型的输入做分词，再去倒排索引搜索
        },
        "country": {
          "type": "keyword"
        },
        "age": {
          "type": "integer"
        },
        "date": {
          "type": "date",
          "format": "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis"
        }
      }
    }
  }
}

解析：

1、index: 
analyzed : 全文 full text
not_analyzed : 精准匹配 exact value
no ：不索引

2、analyzer vs search_analyzer
在索引时，只会去看字段有没有定义 analyzer，有定义的话就用定义的，没定义就用ES预设的

在查询时，会先去看字段有没有定义search_analyzer，如果没有定义，就去看有没有analyzer，再没有定义，才会去使用ES预设的

3、type  （参考自： https://blog.csdn.net/ZYC88888/article/details/83059040）
（1）text: 设置text类型以后，字段内容会被分析，在生成倒排索引以前，字符串会被分析器分成一个一个词项。text类型的字段不用于排序，很少用于聚合（termsAggregation除外）。
（2）keyword: 只能精确匹配到。
（3）数字类型：在满足需求的情况下，尽可能选择范围小的数据类型
（4）Object类型：
PUT my_index/my_type/1
{ 
  "region": "US",
  "manager": { 
    "age":     30,
    "name": { 
      "first": "John",
      "last":  "Smith"
    }
  }
} 

会被索引成 kv 对：
{
  "region":             "US",
  "manager.age":        30,
  "manager.name.first": "John",
  "manager.name.last":  "Smith"
}
对应的mappings:

{
  "mappings": {
    "my_type": { 
      "properties": {
        "region": {
          "type": "keyword"
        },
        "manager": { 
          "properties": {
            "age":  { "type": "integer" },
            "name": { 
              "properties": {
                "first": { "type": "text" },
                "last":  { "type": "text" }
              }
            }
          }
        }
      }
    }
  }
}


（5）date类型
可以指定 date 的格式

(6) Array类型
默认情况下任何字段都可以包含一个或者多个值，但是一个数组中的值要是同一种类型
例如：
字符数组: [ “one”, “two” ]
整型数组：[1,3]
嵌套数组：[1,[2,3]],等价于[1,2,3]
对象数组：[ { “name”: “Mary”, “age”: 12 }, { “name”: “John”, “age”: 10 }]


例如：
PUT /demo_index2/demo_type/1
{
  "properTest": [
    {
      "name": "zhangsan",
      "age": 1
    },
    {
      "name": "lisi",
      "age": 2
    }
  ],
  "properTest2" : "hhhhh"
}

其对应的mapping 如下：
{
  "demo_index2": {
    "mappings": {
      "demo_type": {
        "properties": {
          "properTest": {
            "properties": {
              "age": {
                "type": "long"
              },
              "name": {
                "type": "text",
                "fields": {
                  "keyword": {
                    "type": "keyword",
                    "ignore_above": 256
                  }
                }
              }
            }
          },
          "properTest2": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          }
        }
      }
    }
  }
}

(7) binary类型
接受base64编码的字符串，默认不存储也不可搜索。
(8) ip类型
该类型字段用于存储IPV4或者IPV6的地址。

（9）nested类型
特殊类型的object。普通的object 会将数据打平，这样其不同字段之间的关系也就丢失了，nested 可以使这种关系保留。
例如：

PUT my_index/my_type/1
{
  "group" : "fans",
  "user" : [ 
    {
      "first" : "John",
      "last" :  "Smith"
    },
    {
      "first" : "Alice",
      "last" :  "White"
    }
  ]

会进行打平：
{
  "group" :        "fans",
  "user.first" : [ "alice", "john" ],
  "user.last" :  [ "smith", "white" ]
}

user.first和user.last会被平铺为多值字段，Alice和White之间的关联关系会消失。上面的文档会不正确的匹配以下查询(虽然能搜索到,实际上不存在Alice Smith)：


使用nested字段类型解决Object类型的不足：

PUT my_index
{
  "mappings": {
    "my_type": {
      "properties": {
        "user": {
          "type": "nested" 
        }
      }
    }
  }
}
 
PUT my_index/my_type/1
{
  "group" : "fans",
  "user" : [
    {
      "first" : "John",
      "last" :  "Smith"
    },
    {
      "first" : "Alice",
      "last" :  "White"
    }
  ]
}
 
GET my_index/_search
{
  "query": {
    "nested": {
      "path": "user",
      "query": {
        "bool": {
          "must": [
            { "match": { "user.first": "Alice" }},
            { "match": { "user.last":  "Smith" }} 
          ]
        }
      }
    }
  }
}
 
```



### 1.3 删除

```text
DELETE demo_index2
```

### 1.4 修改-增加字段

> mapping一旦创建之后，就无法修改，只能追加，如果要修改，就需要删除掉整个文档进行重建。

```text
创建时，只增加了properTest properTest2, 如下：
PUT demo_index2
{
    "mappings": {
      "demo_type": {
        "properties": {
          "properTest": {
            "properties": {
              "age": {
                "type": "long"
              },
              "name": {
                "type": "text",
                "fields": {
                  "keyword": {
                    "type": "keyword",
                    "ignore_above": 256
                  }
                }
              }
            }
          },
          "properTest2": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          }
        }
      }
    }
}

现在想添加
PUT /demo_index2/demo_type/_mapping
{
    "properties": {
      "properTest3": {
        "type": "integer"
      }
    }
}

```



### 1.5 修改- 重新建立一个index，然后创建一个新的mapping,通过别名重定向

```text
step1、创建一个索引，这个索引的名称最好带上版本号，比如my_index_v1,my_index_v2等。

step2、创建一个指向本索引的同义词。

Java代码  
curl -XPOST localhost:9200/_aliases -d '  
{  
    "actions": [  
        { "add": {  
            "alias": "my_index",  
            "index": "my_index_v1"  
        }}  
    ]  
}  
'  
 

 此时，你可以通过同义词my_index访问。包括创建索引，删除索引等。

 

step3，需求来了，需要更改mapping了，此时，你需要创建一个新的索引，比如名称叫my_index_v2（版本升级）.，在这个索引里面创建你新的mapping结构。然后，将新的数据刷入新的index里面。在刷数据的过程中，你可能想到直接从老的index中取出数据，然后更改一下格式即可。如何遍历所有的老的index数据，请参考这里。

step4，修改同义词。将指向v1的同义词，修改为指向v2。http接口如下：

Java代码  
curl -XPOST localhost:9200/_aliases -d '  
{  
    "actions": [  
        { "remove": {  
            "alias": "my_index",  
            "index": "my_index_v1"  
        }},  
        { "add": {  
            "alias": "my_index",  
            "index": "my_index_v2"  
        }}  
    ]  
}  
'  
 step5，删除老的索引。

Java代码  
curl -XDELETE localhost:9200/my_index_v1  
```



### 1.6 dynamic mapping 规则

![](../../.gitbook/assets/image%20%285%29.png)



## 2 文档CRUD

### 2.1 创建

#### 2.1.1 有id 的创建 \(put\)

```text
## 有id 的创建
PUT /demo_index2/demo_type/1
{
  "properTest": [
    {
      "name": "zhangsan",
      "age": 1
    },
    {
      "name": "lisi",
      "age": 2
    }
  ],
  "properTest2" : "hhhhh"
}

## 没有id 的创建

```

#### 2.1.2 没有id 的创建 \(post\)

```text
POST /demo_index2/demo_type/
{
  "properTest": [
    {
      "name": "zhangsan",
      "age": 1
    },
    {
      "name": "lisi",
      "age": 2
    }
  ],
  "properTest2" : "hhhhh"
}

```

### 2.2 查看

```text
GET demo_index2/demo_type/_search     // 全部

GET demo_index2/demo_type/1   // 根据id 查询
```

### 2.3 更新

#### 2.3.1 全量覆盖 【检索并修改它，然后重新索引整个文档】

```text
PUT /demo_index2/demo_type/1
{
  "properTest": [
    {
      "name": "zhangsan",
      "age": 11
    },
    {
      "name": "lisi",
      "age": 22222
    }
  ],
  "properTest2" : "hhhhhaaaa"
}
```

#### 2.3.2 只覆盖修改的字段

> `update` API 必须遵循同样的规则。 从外部来看，我们在一个文档的某个位置进行部分更新。然而在内部， `update` API 简单使用与之前描述相同的 _检索-修改-重建索引_ 的处理过程。 区别在于这个过程发生在分片内部，这样就避免了多次请求的网络开销。

```text
POST demo_index2/demo_type/1/_update
{
  "doc": {
    "properTest2": "hhhhhaaaa22222",
    "properTest": [
      {
        "name": "zhangsan222",
        "age": 11
      },
      {
        "name": "lisi222",
        "age": 22222
      }
    ]
  }
} 
```

### 2.4 删除

```text
DELETE /{index}/{type}/{id}
```

## 参考文献：

mapping 的设置： [https://blog.csdn.net/ZYC88888/article/details/83059040](https://blog.csdn.net/ZYC88888/article/details/83059040)


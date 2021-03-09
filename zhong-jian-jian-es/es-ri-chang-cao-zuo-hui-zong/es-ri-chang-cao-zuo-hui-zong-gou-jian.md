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
      "dynamic": false,  // 不会自动创建索引
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
（1）text: 设置text类型以后，字段内容会被分析，在生成倒排索引以前，
字符串会被分析器分成一个一个词项。
text类型的字段不用于排序，很少用于聚合（termsAggregation除外）。
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
  }

会进行打平：
{
  "group" :        "fans",
  "user.first" : [ "alice", "john" ],
  "user.last" :  [ "smith", "white" ]
}

user.first和user.last会被平铺为多值字段，Alice和White之间的关联关系会消失。
上面的文档会不正确的匹配以下查询(虽然能搜索到,实际上不存在Alice Smith)：
GET my_index/my_type/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "user.first": "Alice"
          }
        },
        {
          "match": {
            "user.last": "Smith"
          }
        }
      ]
    }
  }
}

结果：
{
  "took": 10,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "failed": 0
  },
  "hits": {
    "total": 1,
    "max_score": 0.51623213,
    "hits": [
      {
        "_index": "my_index",
        "_type": "my_type",
        "_id": "1",
        "_score": 0.51623213,
        "_source": {
          "group": "fans",
          "user": [
            {
              "first": "John",
              "last": "Smith"
            },
            {
              "first": "Alice",
              "last": "White"
            }
          ]
        }
      }
    ]
  }
}


使用nested字段类型解决Object类型的不足：

PUT my_index
{
  "mappings": {
    "my_type": {
      "properties": {
        "user": {
          "type": "nested",
          "properties": {
              "first": {
                "type": "text",
                "fields": {
                  "keyword": {
                    "type": "keyword",
                    "ignore_above": 256
                  }
                }
              },
              "last": {
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
 
 一条结果也搜不到了。
 
 注意：由于嵌套对象 被索引在独立隐藏的文档中，我们无法直接查询它们。
  相应地，我们必须使用 nested 查询 去获取它们：
  GET my_index/_search
{
  "query": {
    "nested": {
      "path": "user",
      "query": {
        "bool": {
          "must": [
            { "match": { "user.first": "John" }},
            { "match": { "user.last":  "Smith" }} 
          ]
        }
      }
    }
  }
}
返回一条，但是如果使用如下查询，则查不到
  GET my_index/_search
{
      "query": {
        "bool": {
          "must": [
            { "match": { "user.first": "John" }},
            { "match": { "user.last":  "Smith" }} 
          ]
        }
      }
 
}
 
```

看一个包含上述知识点的最常见的mapping:

```text
DELETE student

PUT student
{
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 1
  },
  "mappings": {
    "student_type" : {
      "dynamic": false,
      "properties": {
        "name" : {
          "type" : "text",
          "index" : false
        },
        "country":{
          "type": "text"
        },
        "age" : {
          "type": "integer"
        },
        "address" : {
          "type": "nested",
          "properties": {
            "provice": {
              "type" : "text"
            },
            "city" : {
              "type" : "text"
            }
            
          }
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
>   
> 在ES里为已有索引增加一个新字段以后，老的数据并不会自动就拥有了这个新字段，也就不可能给他一个所谓的默认值。ES里的数据都是文档型的，修改一个文档只能是对原文档做更新，也就是只能借助于重新索引的手段。



> 如果想使老数据有新值，如下链接  
> es 对历史数据增加字段，并赋初始值[： https://blog.csdn.net/zhou\_shaowei/article/details/81975476](https://blog.csdn.net/zhou_shaowei/article/details/81975476)  
>
>
> 如果已经有历史数据，且想让新索引在历史数据上生效，可以参考2.5

```text
创建时，只增加了properTest, 如下：
## 创建索引
PUT demo_index2
{
    "mappings": {
      "demo_type": {
        "properties": {
          "properTest": {
            "type": "keyword"
          }
        }
      }
    }
}

## 插入数据 1
PUT demo_index2/demo_type/1
{
  "properTest": "aaaa"
}

GET demo_index2/demo_type/_search

## 增加mapping
PUT /demo_index2/demo_type/_mapping
{
    "properties": {
      "properTest1": {
        "type": "integer"
      }
    }
}
## 插入数据 2
PUT demo_index2/demo_type/2
{
  "properTest": "aaaa",
  "properTest1": 1
}
## 查看，发现1 并没有 properTest1 这个字段。
GET demo_index2/demo_type/_search

## 结合脚本，对历史数据进行更新
POST demo_index2/_update_by_query
{
  "script": {
    "lang": "painless",
    "inline": "if (ctx._source.properTest1 == null) {ctx._source.properTest1= -1}"
  }
}
## 查看，发现1 的 properTest1 这个字段已经被设置为-1 了。
GET demo_index2/demo_type/_search

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
            "alias": "my_index",   // 别名
            "index": "my_index_v1"    // 被别名的索引
        }}  
    ]  
}  
'  
 

 此时，你可以通过同义词my_index访问。包括创建索引，删除索引等。

 

step3，需求来了，需要更改mapping了，此时，你需要创建一个新的索引，比如名称叫my_index_v2（版本升级）.，
在这个索引里面创建你新的mapping结构。然后，将新的数据刷入新的index里面。
在刷数据的过程中，你可能想到直接从老的index中取出数据，然后更改一下格式即可。
如何遍历所有的老的index数据，请参考这里。

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

### 1.6 跨集群迁移数据

使用 elasticsearch-dump 工具。

### 1.7 dynamic mapping 规则

![](../../.gitbook/assets/image%20%287%29.png)



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

### 2.5 历史数据没有索引，加索引之后，如果让历史数据也支持索引？

```text
DELETE  test_index2

# 创建一个没有 commodity_name  的mapping index, "dynamic"设置为"false"
PUT test_index2
{
	"mappings": {
		"test_type": {
		  "dynamic": "false",
			"properties": {
				"commodity_id": {
					"type": "long"
				},
		
				"picture_url": {
					"type": "keyword"
				}
			}
		}
	}
}

# 插入数据
PUT test_index2/test_type/1
{
  "commodity_id": 1,
  "commodity_name": {
    "desc" : "my name is zhangpeng"
  },
  "picture_url": "www.1.com"
}

# 查询，此时不会返回值。
GET test_index2/test_type/_search
{
  "query": {
    "match": {
      "commodity_name.desc": "zhangpeng"
    }
  }
}

# 更新mapping, 对 commodity_name 的 desc 加索引
PUT test_index2/_mapping/test_type
{

      "properties": {
        "commodity_id": {
          "type": "long"
        },
        "commodity_name": {
          "properties": {
            "desc": {
              "type": "text",
              "index": true
            }
          }
        },
        "picture_url": {
          "type": "keyword"
        }
      }
   
}


# 再查，历史数据还是没有返回
GET test_index2/test_type/_search
{
  "query": {
    "match": {
      "commodity_name.desc": "zhangpeng"
    }
  }
}


# 基于某些条件，把历史数据查出来，然后使用script 更新某个字段（通常是时间戳等跟业务无关的字段）
POST  test_index2/test_type/_update_by_query
{
  "script": {
    "inline": "ctx._source.commodity_id  += 1"
    
  },
  "query" : {
    "match": {
      "commodity_id" : 1
    } 
  }
}

## https://stackoverflow.com/questions/36589963/elasticsearchuse-script-to-update-nested-field

## 再次查询 ，就已经有值返回了。

GET test_index2/test_type/_search
{
  "query": {
    "match": {
      "commodity_name.desc": "zhangpeng"
    }
  }
}

```

注意： 脚本更新，如果命中数据量比较大，则性能会比较差，如下链接有一个可以优化的方案：  
[https://elasticsearch.cn/article/13548](https://elasticsearch.cn/article/13548)

## 参考文献：

mapping 的设置： [https://blog.csdn.net/ZYC88888/article/details/83059040](https://blog.csdn.net/ZYC88888/article/details/83059040)


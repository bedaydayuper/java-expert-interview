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
    "settings": {
        "number_of_shards": 3,
        "number_of_replicas": 1
    },
    "mappings": {
            "properties": {
                "name": {
                    "type":"text"
                },
                "country": {
                    "type":"keyword"
                },
                "age": {
                    "type":"integer"
                },
                "date": {
                    "type": "date",
                    "format": "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis"
            }
        }
    }
}

解析：

```

## 2 文档CRUD


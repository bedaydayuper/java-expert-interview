# MySQL查询优化（三）explain

## 1 概述

![](../.gitbook/assets/image%20%28174%29.png)

如下分析基于 下面结构的表 s1 s2。

```text
CREATE TABLE single_table (
    id INT NOT NULL AUTO_INCREMENT,
    key1 VARCHAR(100),
    key2 INT,
    key3 VARCHAR(100),
    key_part1 VARCHAR(100),
    key_part2 VARCHAR(100),
    key_part3 VARCHAR(100),
    common_field VARCHAR(100),
    PRIMARY KEY (id),
    KEY idx_key1 (key1),
    UNIQUE KEY idx_key2 (key2),
    KEY idx_key3 (key3),
    KEY idx_key_part(key_part1, key_part2, key_part3)
) Engine=InnoDB CHARSET=utf8;
```

## 2  执行计划输出中各列详解

### 2.1 table

EXPLAIN语句输出的每条记录都对应着某个单表的访问方法，该条记录的table列代表着该表的表名。

即使有多个表连表查询，也是每行有一个表，每行记录着一个表的查询方法。

### 2.2 id

1、连接查询的执行计划中，每个表都会对应一条记录，这些记录的id列的值是相同的，出现在前边的表表示驱动表，出现在后边的表表示被驱动表。

2、对于包含子查询的查询语句来说，就可能涉及多个`SELECT`关键字，所以在包含子查询的查询语句的执行计划中，每个`SELECT`关键字都会对应一个唯一的`id`值。



但是这里大家需要特别注意，查询优化器可能对涉及子查询的查询语句进行重写，从而转换为连接查询。

3、包含`UNION`子句的查询语句来说，每个`SELECT`关键字对应一个`id`值

### 2.3 select\_type

#### 2.3.1 SIMPLE

查询语句中不包含`UNION`或者子查询的查询都算作是`SIMPLE`类型

#### 2.3.2 PRIMARY

对于包含`UNION`、`UNION ALL`或者子查询的大查询来说，它是由几个小查询组成的，其中最左边的那个查询的`select_type`值就是`PRIMARY`

#### 2.3.3 UNION

对于包含`UNION`或者`UNION ALL`的大查询来说，它是由几个小查询组成的，其中除了最左边的那个小查询以外，其余的小查询的`select_type`值就是`UNION`

#### 2.3.4`UNION RESULT` 

#### `MySQL`选择使用临时表来完成`UNION`查询的去重工作，针对该临时表的查询的`select_type`就是`UNION RESULT`



#### 2.3.5 ``SUBQUERY

如果包含子查询的查询语句不能够转为对应的`semi-join`的形式，并且该子查询是不相关子查询，并且查询优化器决定采用将该子查询物化的方案来执行该子查询时，该子查询的第一个`SELECT`关键字代表的那个查询的`select_type`就是`SUBQUERY`



#### 2.3.6 DEPENDENT SUBQUERY

如果包含子查询的查询语句不能够转为对应的`semi-join`的形式，并且该子查询是相关子查询，则该子查询的第一个`SELECT`关键字代表的那个查询的`select_type`就是`DEPENDENT SUBQUERY`

#### 2.3.7 `DEPENDENT UNION` 在包含`UNION`或者`UNION ALL`的大查询中，如果各个小查询都依赖于外层查询的话，那除了最左边的那个小查询之外，其余的小查询的`select_type`的值就是`DEPENDENT UNION`



#### 2.3.8 DERIVED

对于采用物化的方式执行的包含派生表的查询，该派生表对应的子查询的`select_type`就是`DERIVED`

#### 2.3.9 MATERIALIZED

  
当查询优化器在执行包含子查询的语句时，选择将子查询物化之后与外层查询进行连接查询时，该子查询对应的`select_type`属性就是`MATERIALIZED`

### 2.4 partitions

// 暂时忽略。

### 2.5 type

对某个表的执行查询时的访问方法

#### 2.5.1 system

当表中只有一条记录并且该表使用的存储引擎的统计数据是精确的，比如MyISAM、Memory，那么对该表的访问方法就是`system`

#### 2.5.2  const

当我们根据主键或者唯一二级索引列与常数进行等值匹配时，对单表的访问方法就是`const`

#### 2.5.3  eq\_ref

在连接查询时，如果`被驱动表`是通过主键或者唯一二级索引列等值匹配的方式进行访问的（如果该主键或者唯一二级索引是联合索引的话，所有的索引列都必须进行等值比较），则对该被驱动表的访问方法就是`eq_ref`

#### 2.5.4  ref

当通过普通的二级索引列与常量进行等值匹配时来查询某个表，那么对该表的访问方法就可能是`ref`

#### 2.5.5  fulltext

// 没有用过，跳过。

#### 2.5.6  ref\_or\_null

当对普通二级索引进行等值匹配查询，该索引列的值也可以是`NULL`值时，那么对该表的访问方法就可能是`ref_or_null`

```text
EXPLAIN SELECT * FROM s1 WHERE key1 = 'a' OR key1 IS NULL;
```

#### 2.5.7  index\_merge

一般情况下对于某个表的查询只能使用到一个索引，但我们唠叨单表访问方法时特意强调了在某些场景下可以使用`Intersection`、`Union`、`Sort-Union`这三种索引合并的方式来执行查询.

```text
mysql> EXPLAIN SELECT * FROM s1 WHERE key1 = 'a' OR key3 = 'a';
+----+-------------+-------+------------+-------------+-------------------+-------------------+---------+------+------+----------+---------------------------------------------+
| id | select_type | table | partitions | type        | possible_keys     | key               | key_len | ref  | rows | filtered | Extra                                       |
+----+-------------+-------+------------+-------------+-------------------+-------------------+---------+------+------+----------+---------------------------------------------+
|  1 | SIMPLE      | s1    | NULL       | index_merge | idx_key1,idx_key3 | idx_key1,idx_key3 | 303,303 | NULL |   14 |   100.00 | Using union(idx_key1,idx_key3); Using where |
+----+-------------+-------+------------+-------------+-------------------+-------------------+---------+------+------+----------+---------------------------------------------+
1 row in set, 1 warning (0.01 sec)
```

从执行计划的`type`列的值是`index_merge`

\`\`

#### 2.5.8 unique\_subquery

类似于两表连接中被驱动表的`eq_ref`访问方法，`unique_subquery`是针对在一些包含`IN`子查询的查询语句中，如果查询优化器决定将`IN`子查询转换为`EXISTS`子查询，而且子查询可以使用到主键进行等值匹配的话，那么该子查询执行计划的`type`列的值就是`unique_subquery`

#### 2.5.9 index\_subquery

`index_subquery`与`unique_subquery`类似，只不过访问子查询中的表时使用的是普通的索引.

#### 2.5.10 range

如果使用索引获取某些`范围区间`的记录，那么就可能使用到`range`访问方法

#### 2.5.11 index

当我们可以使用索引覆盖，但需要扫描全部的索引记录时，该表的访问方法就是`index`

#### 2.5.12 ALL

全表扫描



### 2.6 possible\_keys和key



### 2.7 key\_len



### 2.8 ref



### 2.9 rows



### 2.10 filtered





## 3 


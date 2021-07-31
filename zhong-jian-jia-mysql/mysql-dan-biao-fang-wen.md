# MySQL 单表访问

0、一个例子

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

## 1 访问方法（access method）的概念

MySQL 执行查询语句的方式 成为 访问方法。

同一条语句有多种访问方法，都能得到相同的结果，但是性能有千差万别。

### 1.1 const

1、把这种通过主键或者唯一二级索引列来定位一条记录的访问方法定义为：`const`，意思是常数级别的，代价是可以忽略不计的。不过这种`const`访问方法只能在`主键列`或者\``唯一二级索引列和一个常数进行等值比较时`才有效。  注意是唯一二级索引`，`uniq。跟ref的区别就是只返回了一条数据。

![](../.gitbook/assets/image%20%28166%29.png)



2、唯一二级索引列：

```text
SELECT * FROM single_table WHERE key2 = 3841;
```

第一步 根据二级索引列找到id, 第二个根据id 并利用聚簇索引 找到所有列的值。

3、对于唯一二级索引来说，查询该列为`NULL`值的情况比较特殊，比如这样：

```text
SELECT * FROM single_table WHERE key2 IS NULL;
```

因为唯一二级索引列并不限制 NULL 值的数量，所以上述语句可能访问到多条记录，也就是说 上边这个语句不可以使用`const`访问方法来执行。

### 1.2 ref

1、把这种搜索条件为二级索引列与常数等值比较，采用二级索引来执行查询的访问方法称为：`ref`

![](../.gitbook/assets/image%20%28167%29.png)

对于普通的二级索引来说，通过索引列进行等值比较后可能匹配到**多条**连续的记录，而不是像主键或者唯一二级索引那样最多只能匹配**1条**记录，所以这种`ref`访问方法比`const`差了那么一丢丢，但是在二级索引等值比较时匹配的记录数较少时的效率还是很高的。

2、注意如下情况：

（1）二级索引列值为`NULL`的情况

不论是普通的二级索引，还是唯一二级索引，它们的索引列对包含`NULL`值的数量并不限制，所以我们采用`key IS NULL`这种形式的搜索条件最多只能使用`ref`的访问方法，而不是`const`的访问方法。

（2）对于某个包含多个索引列的二级索引来说，只要是最左边的连续索引列是与常数的等值比较就可能采用`ref`的访问方法

```text
SELECT * FROM single_table WHERE key_part1 = 'god like';

SELECT * FROM single_table WHERE key_part1 = 'god like' AND key_part2 = 'legendary';

SELECT * FROM single_table WHERE key_part1 = 'god like' AND key_part2 = 'legendary' AND key_part3 = 'penta kill';
```



但是如果最左边的连续索引列并不全部是等值比较的话，它的访问方法就不能称为`ref`了，比方说这样：

```text
SELECT * FROM single_table WHERE key_part1 = 'god like' AND key_part2 > 'legendary';
```

### 1.3 ref\_or\_null

## 想找出某个二级索引列的值等于某个常数的记录，还想把该列的值为`NULL`的记录也找出来，就像下边这个查询：

```text
SELECT * FROM single_table WHERE key1 = 'abc' OR key1 IS NULL;
```

当使用二级索引而不是全表扫描的方式执行该查询时，这种类型的查询使用的访问方法就称为`ref_or_null.` 

![](../.gitbook/assets/image%20%28165%29.png)



### 1.4 range

这种利用索引进行范围匹配的访问方法称之为：`range`

```text
SELECT * FROM single_table WHERE key2 IN (1438, 6328) OR (key2 >= 38 AND key2 <= 79);
```

像 in 是单点区间， 大于小于时连续区间。

![](../.gitbook/assets/image%20%28164%29.png)

### 1.5  index

适合**利用了索引树中的非最左列，而且结果包含在联合索引中，不需要回表的情况**



```text
SELECT key_part1, key_part2, key_part3 FROM single_table WHERE key_part2 = 'abc';
```



由于`key_part2`并不是联合索引`idx_key_part`最左索引列，所以我们无法使用`ref`或者`range`访问方法来执行这个语句。但是这个查询符合下边这两个条件：

* 它的查询列表只有3个列：`key_part1`, `key_part2`, `key_part3`，而索引`idx_key_part`又包含这三个列。
* 搜索条件中只有`key_part2`列。这个列也包含在索引`idx_key_part`中。

可以直接通过遍历`idx_key_part`索引的叶子节点的记录来比较`key_part2 = 'abc'`这个条件是否成立，把匹配成功的二级索引记录的`key_part1`, `key_part2`, `key_part3`列的值直接加到结果集中就行了。由于二级索引记录比聚簇索记录小的多（聚簇索引记录要存储所有用户定义的列以及所谓的隐藏列，而二级索引记录只需要存放索引列和主键），而且这个过程也不用进行回表操作，所以直接遍历二级索引比直接遍历聚簇索引的成本要小很多。

### 1.6 all

使用全表扫描。

## 2 注意事项

### 2.1 重温二级索引 + 回表

如果有多个条件，可能这些条件会满足不同的索引，查询优化器会使用最优的条件，然后其余条件在回表时作为过滤条件使用。

### 2.2 明确range访问方法使用的范围区间

当我们想使用`range`访问方法来执行一个查询语句时，重点就是找出该查询可用的索引以及这些索引对应的范围区间。

**在为某个索引确定范围区间的时候只需要把用不到相关索引的搜索条件替换为true**

下面两个例子很有意思：

第一个： and

```text
SELECT * FROM single_table WHERE key2 > 100 AND common_field = 'abc'
```

因为 common\_field 是用不了 索引，所以替换之后是

```text
SELECT * FROM single_table WHERE key2 > 100 AND true;
```

再进一步简化：

```text
SELECT * FROM single_table WHERE key2 > 100
```

说最上边那个查询使用`idx_key2`的范围区间就是：`(100, +∞)`。

第二个： or

```text
SELECT * FROM single_table WHERE key2 > 100 AND common_field = 'abc'

```

\*\*\*\*

因为 common\_field 是用不了 索引，所以替换之后是

```text
SELECT * FROM single_table WHERE key2 > 100 OR true;
```

再进一步简化，就成了

```text
SELECT * FROM single_table WHERE true;

```

。。。。索引不能用了。。。。

这也就说说明如果我们强制使用`idx_key2`执行查询的话，对应的范围区间就是`(-∞, +∞)`，也就是需要将全部二级索引的记录进行回表，这个代价肯定比直接全表扫描都大了。

### **2.3 复杂搜索条件下找出范围匹配的区间**

**第一步 先分析条件中有几个列 可以使用索引**

**第二步 然后对每种可能使用索引的情况进行如下规则分析：**

**在为某个索引确定范围区间的时候只需要把用不到相关索引的搜索条件替换为true**

第三步： 比较第二步 分析之后的结果，看最后使用哪个索引。

```text
SELECT * FROM single_table WHERE 
        (key1 > 'xyz' AND key2 = 748 ) OR
        (key1 < 'abc' AND key1 > 'lmn') OR
        (key1 LIKE '%suf' AND key1 > 'zzz' AND (key2 < 8000 OR common_field = 'abc')) ;
这个搜索条件真是绝了，不过大家不要被复杂的表象迷住了双眼，按着下边这个套路分析一下：

1 首先查看WHERE子句中的搜索条件都涉及到了哪些列，哪些列可能使用到索引。

这个查询的搜索条件涉及到了key1、key2、common_field这3个列，
然后key1列有普通的二级索引idx_key1，key2列有唯一二级索引idx_key2。

对于那些可能用到的索引，分析它们的范围区间。

(1)假设我们使用idx_key1执行查询

我们需要把那些用不到该索引的搜索条件暂时移除掉，移除方法也简单，直接把它们替换为TRUE就好了。上边的查询中除了有关key2和common_field列不能使用到idx_key1索引外，key1 LIKE '%suf'也使用不到索引，所以把这些搜索条件替换为TRUE之后的样子就是这样：

(key1 > 'xyz' AND TRUE ) OR
(key1 < 'abc' AND key1 > 'lmn') OR
(TRUE AND key1 > 'zzz' AND (TRUE OR TRUE))
化简一下上边的搜索条件就是下边这样：

(key1 > 'xyz') OR
(key1 < 'abc' AND key1 > 'lmn') OR
(key1 > 'zzz')
替换掉永远为TRUE或FALSE的条件

因为符合key1 < 'abc' AND key1 > 'lmn'永远为FALSE，所以上边的搜索条件可以被写成这样：

(key1 > 'xyz') OR (key1 > 'zzz')
继续化简区间

key1 > 'xyz'和key1 > 'zzz'之间使用OR操作符连接起来的，意味着要取并集，所以最终的结果化简的到的区间就是：key1 > xyz。也就是说：上边那个有一坨搜索条件的查询语句如果使用 idx_key1 索引执行查询的话，需要把满足key1 > xyz的二级索引记录都取出来，然后拿着这些记录的id再进行回表，得到完整的用户记录之后再使用其他的搜索条件进行过滤。

(2)假设我们使用idx_key2执行查询

我们需要把那些用不到该索引的搜索条件暂时使用TRUE条件替换掉，
其中有关key1和common_field的搜索条件都需要被替换掉，替换结果就是：

(TRUE AND key2 = 748 ) OR
(TRUE AND TRUE) OR
(TRUE AND TRUE AND (key2 < 8000 OR TRUE))
哎呀呀，key2 < 8000 OR TRUE的结果肯定是TRUE呀，也就是说化简之后的搜索条件成这样了：

key2 = 748 OR TRUE
这个化简之后的结果就更简单了：

TRUE
这个结果也就意味着如果我们要使用idx_key2索引执行查询语句的话，
需要扫描idx_key2二级索引的所有记录，然后再回表，这不是得不偿失么，
所以这种情况下不会使用idx_key2索引的。


```

\*\*\*\*

## 3 索引合并

把使用到多个索引来完成一次查询的执行方法称为index merge。

### 3.1 **Intersection合并**

**1、交集。**

某个查询可以使用多个二级索引，将从多个二级索引中查询到的结果取交集

2、（1）可以使用一个索引，然后回表； （2）也可以使用多个索引查出主键，然后取主键的交集，最后回表。 

至于使用哪种情况，取决于查询优化器。

3、

`MySQL`在某些特定的情况下才可能会使用到`Intersection`索引合并：

* 情况一：二级索引列是等值匹配的情况，对于联合索引来说，在联合索引中的每个列都必须等值匹配，不能出现只匹配部分列的情况。
* 情况二：主键列可以是范围匹配

满足如上两种情况，只是intersection 的必要条件，但是不是充分条件。就是说即使情况一、情况二成立，也不一定发生`Intersection`索引合并，这得看优化器的心情。

### **3.2 Union合并**

`1、Union`是并集的意思，适用于使用不同索引的搜索条件之间使用`OR`连接起来的情况。

2、MySQL在某些特定的情况下才可能会使用到`Union`索引合并：

* 情况一：二级索引列是等值匹配的情况，对于联合索引来说，在联合索引中的每个列都必须等值匹配，不能出现只出现匹配部分列的情况。
* 情况二：主键列可以是范围匹配
* 情况三：使用`Intersection`索引合并的搜索条件。就是搜索条件的某些部分使用`Intersection`索引合并的方式得到的主键集合和其他方式得到的主键集合取交集

```text
SELECT * FROM single_table WHERE key_part1 = 'a' AND key_part2 = 'b' AND key_part3 = 'c' OR (key1 = 'a' AND key3 = 'b');

```

当然，查询条件符合了这些情况也不一定就会采用`Union`索引合并，也得看优化器的心情。优化器只有在单独根据搜索条件从某个二级索引中获取的记录数比较少，通过`Union`索引合并后进行访问的代价比全表扫描更小时才会使用`Union`索引合并。

### **3.3 Sort-Union合并**

```text
SELECT * FROM single_table WHERE key1 < 'a' OR key3 > 'z'
```

* 先根据`key1 < 'a'`条件从`idx_key1`二级索引中获取记录，并按照记录的主键值进行排序
* 再根据`key3 > 'z'`条件从`idx_key3`二级索引中获取记录，并按照记录的主键值进行排序
* 因为上述的两个二级索引主键值都是排好序的，剩下的操作和`Union`索引合并方式就一样了。

把上述这种先按照二级索引记录的主键值进行排序，之后按照`Union`索引合并方式执行的方式称之为`Sort-Union`索引合并，很显然，这种`Sort-Union`索引合并比单纯的`Union`索引合并多了一步对二级索引记录的主键值排序的过程。



Sort-Union的适用场景是单独根据搜索条件从某个二级索引中获取的记录数比较少，这样即使对这些二级索引记录按照主键值进行排序的成本也不会太高。

### **3.4 索引合并注意事项**

#### **3.4.1 联合索引替代Intersection索引合并**

```text
SELECT * FROM single_table WHERE key1 = 'a' AND key3 = 'b';
```

可以单独使用key1 的索引，也可以单独使用key3的索引， 也可以使用 key1/key3的**Intersection 索引， 当然，也可以为干掉 key1 key3 两个单独的索引，建一个联合索引。**


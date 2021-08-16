# MySQL事务简介

## 1 事务的特性

### 1.1 A 原子性

要么全做，要么全不做。

### 1.2 C 一致性

数据库世界只是现实世界的一个映射，现实世界中存在的约束当然也要在数据库世界中有所体现。如果数据库中的数据全部符合现实世界中的约束（all defined rules），我们说这些数据就是一致的，或者说符合`一致性`的。

### 1.3 I 隔离性

保证其他状态转换不会影响本次状态转换。

### 1.4 D 持久性 Durability

一个状态转换完成后，转换的结果将会被永久的保留。这个规则叫做持久性。

## 2 事务的概念

### 2.1 事务的状态

![image-20210816104657292](file:///Users/zhangpeng/Library/Application%20Support/typora-user-images/image-20210816104657292.png?lastModify=1629084116)

```text
活跃的：正在执行中
部分提交的：事务最后一个操作执行完成，但是由于操作都在内存中执行，此时没有有刷新到磁盘上
失败的：当事务处于活动的或者部分提交的 状态时，出错了。
中止的：事务失败之后，需要回滚。回滚完毕时处于 中止状态。
提交的：处于  “部分提交的”状态的事务 同步到磁盘之后，处于 `提交的`状态。
```

## 3 MySQL 中事务的语法

### 3.1 开启

BEGIN 或者 START TRANSACITON.

### 3.2 提交

COMMIT

### 3.3 手动中止

ROLLBACK

`ROLLBACK`语句是我们程序员手动的去回滚事务时才去使用的，如果事务在执行过程中遇到了某些错误而无法继续执行的话，事务自身会自动的回滚.

### 3.4 自动提交

通过 `Show VARIABLES LIKE 'autocomit'` 查看 是否开启了自动提交。

### 3.5 隐式提交

关闭了自动提交，但是如下场景下会隐式提交：

```text
1、定义或修改数据库对象的数据定义语言（Data definition language，缩写为：DDL）。
2、隐式使用或修改mysql数据库中的表
3、事务控制或关于锁定的语句。
当我们在一个事务还没提交或者回滚时就又使用START TRANSACTION或者BEGIN语句开启了另一个事务时，会隐式的提交上一个事务
4、加载数据的语句
LOAD DATA
5、关于MySQL复制的一些语句
使用START SLAVE、STOP SLAVE、RESET SLAVE、CHANGE MASTER TO等语句时也会隐式的提交前边语句所属的事务。
6、其它的一些语句
使用ANALYZE TABLE、CACHE INDEX、CHECK TABLE、FLUSH、 LOAD INDEX INTO CACHE、OPTIMIZE TABLE、REPAIR TABLE、RESET等语句也会隐式的提交前边语句所属的事务。
​
```

### 3.6 保存点

在事务对应的数据库语句中打几个点，我们在调用`ROLLBACK`语句时可以指定会滚到哪个点，而不是回到最初的原点。

```text
RELEASE SAVEPOINT 保存点名称;
```


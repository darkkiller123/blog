---
title: mysql隔离等级
---

## 1 储存引擎

### 查看当前支持的储存引擎

``` 
show engines;
```

### 查看默认的储存引擎

``` 
show variables like '%engine%';
```

### 查看表的储存引擎

``` 
show create table XXX;
```


## 2、字符集

### 查看字符集
```
show character set;
等价
select * from information_schema.character_sets;
```

### 查看字符相关的设置

```
show variables like 'char%';
```


## 3、事务并行问题&隔离等级

### 事务的4个特点，ACID
举例子，假设A借给B20块，那么执行以下操作。A先减少20块，B再加20块。

[^_^]: # 原一隔持

- 1、A Atomicity 原子性，意思就是操作要么完成，要么都失败，不会出现减少了钱，却没有给B还钱的情况。
- 2、C Consistency 一致性，意思是其他事务看到的情况，只会看到这次借钱成功或者借钱失败两种情况，不可能看到A减少了20块，B还没加20块的【中间状态】。
- 3、I Isolation 隔离性，意思可以多个事务并发操作，不会互相影响，发生死锁之类。
- 4、D Durability 持久性，意思是一旦成功，就是永久性的，就算宕机也不会受影响。


### 【并发时】，事务带来的问题
- 1、脏读，指在数据库访问中，事务T1将某一值修改，然后事务T2读取该值，此后T1因为某种原因撤销对该值的修改，这就导致了T2所读取到的数据是无效的。
- 2、不可重复读：一个事务范围内两个相同的查询却返回了不同数据。(针对update、delete)
- 3、幻读：在一个事务中,两次查询的结果不一致(针对的insert操作，莫名其妙多了一条记录) 
- 4、不可重复读和幻读的区别：https://www.cnblogs.com/itcomputer/articles/5133254.html

### 关系表
| 隔离级别    |      脏读（Dirty Read）   | 不可重复读（NonRepeatable Read）     | 幻读（Phantom Read） |
| :----------- | 		:---------- | :---------- | :----------|
|未提交读（Read uncommitted） | 可能  |  可能 | 可能|
|已提交读（Read committed）   | 不可能  |  可能 | 可能|
|可重复读（Repeatable read） | 不可能  |  不可能 | 可能|
|可串行化（Serializable ）  | 不可能  |  不可能 | 不可能|



## 4、隔离级别

### 查看当前隔离等级
```
show variables like '%isolation%';
select @@tx_isolation;
结果：REPEATABLE-READ
```

### 读未提交级别
```
set  tx_isolation='read-uncommitted';
select @@tx_isolation;

# 事务A
start transaction;
select * from tx;

# 事务B
update tx set num=10 where id=1;
select * from tx;

# 事务A
select * from tx;

结果：A读取了B未提交的update
``` 

### 读已提交级别
```
set  tx_isolation='read-committed';
select @@tx_isolation;

# 事务A
start transaction;
select * from tx;

# 事务B
update tx set num=10 where id=1;
commit;
select * from tx;

# 事务A
select * from tx;

结果：A前后读取的数据不一致，不可重复读
``` 

### 可重复读隔离级别
```
set  tx_isolation='repeatable-read';
select @@tx_isolation;

# 事务A
start transaction;
select * from tx;

# 事务B
insert into tx values(4,4);
commit;
select * from tx;

# 事务A
select * from tx;


结果：innodb存储引擎是有mvcc多版本控制,所以看不到最后insert数据的时候,本是可以看到事务B更新的数据,所以没有出现幻读现象
``` 



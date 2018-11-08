---
title: mysql 命令
---
mysql 常用命令总结。

## 默认表结构

```
CREATE TABLE `user_profile` (
    `uid` int(11) unsigned NOT NULL COMMENT 'uid',
    `nickname` varchar(20) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '昵称',
    `gender` int(10) NOT NULL DEFAULT '0' COMMENT '性别 1男 2女 默认没性别',
    `avatar` varchar(512) NOT NULL DEFAULT '' COMMENT '头像',
    PRIMARY KEY (`uid`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```
<br/>

## 0、调试&其他

### 查询调试

``` bash
explain select * from user_profile;
```
<br/>

## 1、查询相关

### 去重查询之distinct

假设我们要简单地去重，可以使用mysql的distinct函数,例子如下:

```
select distinct(nickname) from user_profile;

select nickname from user_profile;

返回结果：31 行
```

但是distinct 函数是有问题的。比如我们有需求，要获取唯一昵称同时，要获取uid。那么查询语句如下：

```
select distinct(nickname), uid from user_profile;

返回结果：4381 行

```

发现了吗，上面的查询基本等价于以下查询：

```
select * from user_profile;

返回结果：4381 行

```

### 结论
- 1、distinct只能用于简单的去重，无法去重+获取多个字段
- 2、如果需要去重同时，获取两个字段，那么就只能使用group
```
select * from user_profile group by nickname;

返回结果：31 行

```
<br/>

## 2、排序
- 1、默认升序
- 2、加关键字desc,可以降序排序。order by uid desc

```
select * from user_profile order by uid;
```

## 3、安全更新，sql_safe_updates
- 执行不带where的sql时候，可能会引发重大的安全问题，
- 可以通过sql_safe_updates参数来禁止执行不带where的update操作

```
开启禁用命令
set sql_safe_updates=1;

```
<br/>

## 4、模糊搜索

### 对比较简单的需求，我们可以用mysql 通配符，like, not like

| 字段    | 说明                       |
| :----------- | :----------------------- |
| % | 替代 0 个或多个字符 |
| - | 替代一个字符 |
| [charlist] | 字符列中的任何单一字符 |
| [^charlist] 或 [!charlist] | 不在字符列中的任何单一字符  |

### 对于复杂的需求，我们只能用 mysql 扩展的正规表达式匹配, REGEXP, not REGEXP
| 字段    | 说明                       |
| :----------- | :----------------------- |
| ^ | 匹配输入字符串的开始位置。如果设置了 RegExp 对象的 Multiline 属性，^ 也匹配 '\n' 或 '\r' 之后的位置。|
| $ | 匹配输入字符串的结束位置。如果设置了RegExp 对象的 Multiline 属性，$ 也匹配 '\n' 或 '\r' 之前的位置。|
| . | 匹配除 "\n" 之外的任何单个字符。要匹配包括 '\n' 在内的任何字符，请使用象 '[.\n]' 的模式。 |
| [...]  | 字符集合。匹配所包含的任意一个字符。例如， '[abc]' 可以匹配 "plain" 中的 'a'。|
| [^...] |负值字符集合。匹配未包含的任意字符。例如， '[^abc]' 可以匹配 "plain" 中的'p'。 |
| p1丨p2丨p3 |匹配 p1 或 p2 或 p3。例如，'z丨food' 能匹配 "z" 或 "food"。'(z丨f)ood' 则匹配 "zood" 或 "food"。 |
| *  |匹配前面的子表达式零次或多次。例如，zo* 能匹配 "z" 以及 "zoo"。* 等价于{0,}。 |
| + |匹配前面的子表达式一次或多次。例如，'zo+' 能匹配 "zo" 以及 "zoo"，但不能匹配 "z"。+ 等价于 {1,}。 |
| {n}  | n 是一个非负整数。匹配确定的 n 次。例如，'o{2}' 不能匹配 "Bob" 中的 'o'，但是能匹配 "food" 中的两个 o。 |
| {n,m} | m 和 n 均为非负整数，其中n <= m。最少匹配 n 次且最多匹配 m 次。 |
<br/>

## 5、连接
- 作用：把来自两个或多个表的行结合起来
- 连接基础；基于相同的字段

### 左连接

A LEFT JOIN B ON XXXX ====> return A 相关的记录, B如果不存在，那么该字段值为NULL

```
select count(1) from user_profile left join user_account on user_profile.uid = user_account.uid;

输出=user_profile 的行数=4381 行
user_profile 的行数: 4381 行
user_account 的行数: 173 行

```

### 右连接

A RIGHT JOIN B ON XXXX ====> return B 相关的记录, A如果不存在，那么该字段值为NULL

```
select count(1) from user_profile right join user_account on user_profile.uid = user_account.uid ;

输出=user_account的行数=173 行
```

注解：
- 1、实际左连接就够用了，将两个table换过来，就是右连接的输出结果
- 2、本质和左连接没啥区别，只是顺序不同

### 内连接,或等值连接 INNER JOIN

A INNER JOIN B ON XXXX ====> return (A 且 B) 相关记录

```
select count(1) from user_profile right join user_account on user_profile.uid = user_account.uid ;

输出=user_account、user_profile 同时存在的记录 =143行
```
<br/>


## 6、NULL 特殊处理

背景1: select 中的 where 不能识别NULL，如以下语句会什么也不输出的。

```
select 1 from user_profile where NULL;
select 1 from user_profile where NOT NULL;
```

背景2：mysql 逻辑处理遇到NULL 时候，会输出NULL。

```
select NULL = 1;
select NULL != 1;
select NULL = NULL;
select NULL != NULL;

全部会输出NULL
```

结论，mysql 中如果存在一些记录字段值为NULL, 这时候不能用 select XXX from XXXXXX where X != NULL;
而应该如下使用：

```
select 1 from user_profile where NULL is NULL;
select 1 from user_profile where NULL is not NULL;
```
<br/>

## 7、UNION 操作符->合并输出用
假设我要执行两条sql，第一条查资料表信息，第二条查账号表信息，我想两条sql一次执行，一次返回。

- 自动去重 union
```
select uid from user_profile
union
select uid from user_account;
```

- 不去重 union all
```
select uid from user_profile
union all
select uid from user_account;
```

<br/>

## 8、事务

### 杂记
- 1、只有Innodb支持事务。
- 2、事务用于管理insert、update、delete，要么都成功，要么都失败。
- 3、命令行模式下，默认事务是自动提交的。可以通过SET AUTOCOMMIT=0 显示关闭自动提交。


### 事务的4个特点，ACID
举例子，假设A借给B20块，那么执行以下操作。A先减少20块，B再加20块。

[^_^]: # 原一隔持

- 1、A Atomicity 原子性，意思就是操作要么完成，要么都失败，不会出现减少了钱，却没有给B还钱的情况。
- 2、C Consistency 一致性，意思是其他事务看到的情况，只会看到这次借钱成功或者借钱失败两种情况，不可能看到A减少了20块，B还没加20块的【中间状态】。
- 3、I Isolation 隔离性，意思允许多个事务并发操作。
- 4、D Durability 持久性，意思是一旦成功，就是永久性的，就算宕机也不会受影响。


### 事务相关命令
```
主要用的：
BEGIN <==> START TRANSACTION 开始一个事务
ROLLBACK <==> COMMIT WORK 事务回滚
COMMIT  <==>  ROLLBACK WORK 事务确认

其他的：
SAVEPOINT identifier 设置保存点，数据库可以有多个保存点。
RELEASE SAVEPOINT identifier 删除指定的保存点
ROLLBACK TO identifier 事务回滚到保存点
SET TRANSACTIO XXX 用来设置事务的隔离级别。

SET AUTOCOMMIT=0 命令行下禁止自动提交
SET AUTOCOMMIT=1 命令行下自动提交
```

### 【并发时】事务带来的问题
- 0、虽然数据库允许多个事务并发操作，但是在不同隔离等级下，会出现不同的问题。
- 1、脏读，指在数据库访问中，事务T1将某一值修改，然后事务T2读取该值，此后T1因为某种原因撤销对该值的修改，这就导致了T2所读取到的数据是无效的。
- 2、不可重复读：一个事务范围内两个相同的查询却返回了不同数据。
- 3、幻读：事务修改以后，还没提交，就被其他事务看到了。

### 脏读、幻读、不可重复读之间的联系
- 待补充

### 并发事务解决方案———使用不同的事务隔离等级
- InnoDB存储引擎提供事务的隔离级别有READ UNCOMMITTED、READ COMMITTED、REPEATABLE READ和SERIALIZABLE。

<br/>

## 9、alter 修改表
- 语法 ALTER TABLE xxxxxx ADD/DROP/MODIFY/CHANGE
- MODIFY 只能用于改类型，不能改名字，而change可以用于改名字

#### 简单的例子说明
```
ALTER TABLE xxxxxx  DROP nickname;   删除列

ALTER TABLE xxxxxx  ADD `nickname` varchar(20) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '昵称'；   

ALTER TABLE xxxxxx  MODIFY `nickname` varchar(20) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '昵称'；

ALTER TABLE xxxxxx  CHANGE `nickname` `nickname` varchar(20) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '昵称'；

```

### alter 的其他用法
- 修改表名
- 修改存储引擎
- 修改索引
- 修改外键


```
修改表名 ALTER TABLE testalter_tbl RENAME TO alter_tbl;

修改存储引擎： alter table tableName engine=myisam;

增加索引 ALTER TABLE 表名 ADD 索引类型 （unique,primary key,fulltext,index）[索引名]（字段名）
//普通索引
alter table table_name add index index_name (column_list) ;
//唯一索引
alter table table_name add unique (column_list) ;
//主键索引
alter table table_name add primary key (column_list) ;
// 删除索引
alter table table_name drop index index_name ;

删除外键约束 alter table tableName drop foreign key 
keyName;

```
<br/>

## 10、索引
- 1、索引本身也就是一张表，该表保存了主键与索引字段，并指向实体表的记录
- 2、会降低查询、更新、删除速度。
- 3、占用一定的磁盘空间

### 创建索引
- alter table table_name add (index | unique | primary key)   index_name (column_list) ;
 - CREATE INDEX indexName ON mytable(username(length)); 
 - 创建表时候创建索引

 ### 删除索引
 - ALTER TABLE testalter_tbl DROP PRIMARY KEY;
 - ALTER TABLE testalter_tbl index index_name;

 
 ### 显示索引信息
 -  SHOW INDEX FROM table_name; 
<br/>
 
 ## 11、临时表
 - 1、我们需要临时表，用于保存一些临时数据。
 - 2、临时表只有当前连接可以看到。
 - 3、当连接断开时候，临时表会被删除，也可以手动删除
 
 ```
 CREATE TEMPORARY TABLE SalesSummary (XXXXXXXXX);
 ```
 <br/>

 ## 12、复制表
 
 ### 方法1
 - 1、首先查看表的结构，然后建立一个新表。
 - 2、INSERT INTO 新表 (字段..) select XXXX from 新表
 
 ### 方法2
 ```
 CREATE TABLE 新表 LIKE 旧表;
 INSERT INTO 新表 SELECT * FROM 旧表;
 ```
 <br/>

 ## 13、AUTO_INCREMENT 自增
 <br/>

 ## 14、重复数据问题
 
 ### 防范
 - 1、设置为主键或者唯一键。 PRIMARY KEY（主键） 或者 UNIQUE（唯一）

 ### 查询重复的记录
``` 
查询出现次数大于1次的记录
SELECT COUNT(*) as repetitions, last_name, first_name FROM person_tbl GROUP BY last_name, first_name HAVING repetitions > 1
```

 ### 删除重复的记录
```
CREATE TABLE tmp SELECT last_name, first_name, sex FROM person_tbl  GROUP BY (last_name, first_name, sex);
DROP TABLE person_tbl;
ALTER TABLE tmp RENAME TO person_tbl;
```

### 删除重复记录(简单版)
```
强制在表中加入索引
ALTER IGNORE TABLE person_tbl ADD PRIMARY KEY (last_name, first_name);
```
 <br/>

## 15、insert 相关的
- INSERT IGNORE INTO 如果存在，插入则无效。
- insert into xxx on duplicate key update XXX 
<br/>

## 16、sql 注入
- 1、限制输入长度
- 2、不用拼装sql，使用参数化的sql或者直接使用存储过程
- 3、不要用管理员权限连接
- 4、应用出错给的错误应该尽可能少
<br/>

## 17、数据库导出

### 方法1
```
select * from user_profile INTO OUTFILE "/tmp/a.out";
```

### 方法2 导出成csv格式
```
select * from user_profile INTO OUTFILE "/tmp/a.out"
FIELDS TERMINATED BY ',' ENCLOSED BY '"'
LINES TERMINATED BY '\r\n';
```

### 方法3 导出成sql脚本
- 1、导出某张表

```
mysqldump -u root -p user_profile user_account > dump.txt
```

- 2、导出某个库
```
mysqldump -u root -p user_profile > dump.txt
```

- 3、备份所有数据库
```
mysqldump -u root -p --all-databases > database_dump.txt
```
<br/>

## 18、数据库导入
- 1、source命令
```
source /tmp/data.sql
```

- 2、LOAD DATA命令
```
LOAD DATA LOCAL INFILE 'dump.txt' INTO TABLE mytbl;
```


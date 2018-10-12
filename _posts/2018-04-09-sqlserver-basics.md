---
layout: post
title: SQL Server基础知识整理
---
## Batch

一个batch对应一个执行计划，如果有编译错误一个batch中的所有语句都不会执行。
- 大部分运行错误组织剩余语句执行。
- 一些运行错误，如违反约束，只组织当前语句。
运行错误之前的语句不受影响。
- CREATE DEFAULT, CREATE FUNCTION, CREATE PROCEDURE, CREATE RULE, CREATE SCHEMA, CREATE TRIGGER, CREATE VIEW必须单独在一个batch中。
- 同一个batch不能修改表，同时引入修改过的列。
- EXECUTE语句如果是batch中的第一个语句，那么EXECUTE关键字可以省略。

## 事务（transaction）

事务是不可分割的最小处理单元。需要满足acid。

### Implicit Transactions

执行某些语句时自动执行begin transaction。

### Autocommit Transactions

默认事务模式。会被Implicit Transactions和explicit transaction覆盖。每一条语句为一个transaction。

### 原子性

要么全部成功，要么全部失败。

### 一致性

事务结束后数据库数据必须和定义的规则保持一致。规则包括数据类型，触发器，各种约束条件。

### 隔离性

隔离性保证同时执行的事务之间不会互相干扰。隔离等级
- Serializable事务串行执行
- Repeatable read事务执行过程中所有读到的数据都持有共享锁。
- SNAPSHOT快照。只能获取到事务开始前所有commit的数据。
- READ COMMITTED无法读取到其他事务已经修改但是没有commit的数据。默认模式。
- READ UNCOMMITTED脏读。

### 持久性

使用数据库备份和事务日志保证提交的数据不会丢失。

## Locking和row versioning
一个事务修改一份数据时，必须持有某种锁直到事务结束以防止其他事务修改同一份数据。
细粒度的锁如行锁增加并行性额外开销大，粗粒度的锁减小并行性额外开销小。
数据库引擎需要获取一系列不同等级的锁才能充分保护资源。叫做lock hierarchy。

### update lock
在repeatable read以上的isolation中两个事务同时持有read锁然后试图修改数据就会发生死锁。update锁和update锁互斥。

### intent lock
代表存在低层级锁。非必需，为了提高性能。

### key-range lock
避免幻象读。保护key-range内的数据不受到任何修改。

### indexed view
物化视图。

#### slowly changing dimension
- type 1 update record
- type 2 create new dimension record
- type 3 create a current value field

### tuning
三种metric：
- CPU使用 影响因素过多不适用
- 执行时间 
- 磁盘使用 代价很高，适合作用评判标准
所以最好使用logical page reads因为phisical page reads不稳定，取决于数据是否在内存中。logical page reads可重复，不受使用环境的影响。

从sys.dm_exec_query_stats取得花销最大的query，Cardinality is simply the number of distinct values that appear in a column. 除非表的数据量很小，要保证join两边的列都有索引。We avoided creating a useless index by examining the cardinality of the data. Cardinality不等于selectivity.
In a table having a clustered index, all nonclustered indexes include the clustering key. As a general rule, you want to make the most selective search argument the first column in a multi-column index. create index include index to cover the query. Make sure the cost of the solution does not outweigh the benefits of optimization.  
Many-to-many merge join速度慢，尽量避免，尽量建立约束，如唯一约束，帮助执行计划优化。

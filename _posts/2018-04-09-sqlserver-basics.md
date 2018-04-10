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


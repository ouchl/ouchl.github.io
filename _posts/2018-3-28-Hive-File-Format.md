---
layout: post
title: Hive文件格式
---

## HiveServer2 

HiveServer2是供客户端连接Hive的服务器，支持多用户并发访问和验证。支持JDBC和ODBC。

## Beeline

Beeline是替换Hive CLI的命令行工具。基于SQLLine CLI的JDBC客户端。  
`!connect jdbc:hive2://localhost:10000 username password`

## Dbvisualizer

设置驱动文件为hive-jdbc-2.3.2-standalone.jar即可。

## 文件格式

### TextFile

文本文件。如CSV文件，JSON文件。

### Sequence File

数据以二进制键值对的形式保存。用来保存Hadoop MapReduce的中间结果。

### RCFile

和Sequence File一样，数据以二进制键值对的形式保存。首先按行将数据分成不同的row group，然后在每一个row group中再按列保存。主要用在Hadoop MapReduce。

### ORCFile

[ORC](https://orc.apache.org/)(Optimized Row Columnar)优化过的文件格式，减少75%的存储空间。比TextFile，Sequence File，RCFile的性能更好，ORC文件包括行数据组和辅助信息文件尾。默认stripe大小250MB。文件尾包括行组列表，每个行组记录条数，每一列的类型。每个行组包括索引数据，行数据和尾。索引数据包括每一列的最大值和最小值以及行位置。
设置：

|-----------------+------------+-----------------|
| 参数  | 默认  |  描述 | 
|---|---|---|
| orc.compress  | ZLIB  | 压缩算法  |
| orc.compress.size  |  262,144 | 压缩块大小  |
| orc.stripe.size  |  67,108,864 |  每个分组块的字节数 |
| orc.row.index.stride | 10,000 | 索引条目间的行数 |
| orc.create.index | true | 是否创建行索引 |
| orc.bloom.filter.columns | "" | bloomfilter列 |
| orc.bloom.filter.fpp | 0.05 | bloomfilter假阳性概率 |
|-----------------+------------+-----------------|

按列存储的优势在于高压缩比，高性能。
提供三种索引，文件级别的列索引，stripe级别的列索引，行级别的列索引。文件和stripe级别的统计信息放在文件footer中，行级别的索引放在包括列统计信息和每一个行组的起始位置。

### Parquet 

[Parquet](http://parquet.apache.org/)Hadoop生态下的按列存储格式。
ORC+ZLIB比Paqruet+Snappy性能高。

|-----------------+------------+-----------------+----------------|
| Default aligned |Left aligned| Center aligned  | Right aligned  |
|-----------------|:-----------|:---------------:|---------------:|
| First body part |Second cell | Third cell      | fourth cell    |
| Second line     |foo         | **strong**      | baz            |
| Third line      |quux        | baz             | bar            |
|-----------------+------------+-----------------+----------------|
| Second body     |            |                 |                |
| 2 line          |            |                 |                |
|=================+============+=================+================|
| Footer row      |            |                 |                |
|-----------------+------------+-----------------+----------------|
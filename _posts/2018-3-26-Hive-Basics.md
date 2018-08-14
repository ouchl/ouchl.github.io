---
layout: post
title: Hive基础知识整理
---

## Hive基本概念

Hive是基于Hadoop的数据仓库。Hive的作用是大数据概括查询分析，可以添加用户自定义函数（UDF）方便数据分析。Hive不好用来处理线上事务，最好用途是传统数据仓库任务。
文件夹对应表。create table定义了查询时的schema。external table与文件夹的管理分开。每一个partition对应一个folder。名字是partition_column='x'。skewed table字段
某一个特殊值和其他值分开存放。CLUSTERED table根据字段的哈希值分开存放，Join时速度更快。partition是伪字段，字段不需要再定义partition字段。同一个folder可以对应不同的table，实现schema的校验功能。

### 数据单元

- Databases数据库：和Schemas同义。作用避免命名冲突，实现安全控制。

>在 SqlServer和Oracle等数据库软件中，Schema是Database的子集，用来加强权限控制。Mysql等对二者不加区分。

- Tables表
- Partitions分区：按照虚拟列的值将表中数据分文件存在，可以加速查询速度。
> SqlServer的做法是先定义分区函数（如何按照数值分成几个区），再定义分区方案sheme（将分区放在哪些filegruop），最后将分区方案应用到表的某一个列上。和Hive的虚拟列不同。

- Buckets桶（Clusters簇）：每一个分区中的数据可以根据表中某一列的哈希值分到不同的bucket里。

### 数据类型

#### 原始类型

- TINYINT 1 byte
- SMALLINT 2 byte
- INT 4 byte
- BIGINT 8 byte
- BOOLEAN
- FLOAT single precision
- DOUBLE double precision
- DECIMAL
- STRING
- VARCHAR
- CHAR
- TIMESTAMP
- DATE
- BINARY

#### 复杂类型

- structs: 可以用.获取成员。c列的类型为STRUCT {a INT; b INT}。c.a获取a的值
- maps: 使用col['member']获取成员。
- arrays: 使用数组下标获取成员。‘

## 示例

####  定义列分隔符

```sql
CREATE TABLE page_view(viewTime INT, userid BIGINT,
                page_url STRING, referrer_url STRING,
                ip STRING COMMENT 'IP Address of the User')
COMMENT 'This is the page view table'
PARTITIONED BY(dt STRING, country STRING)
ROW FORMAT DELIMITED
        FIELDS TERMINATED BY '1'
STORED AS SEQUENCEFILE;
```

> 无法修改行分隔符，因为行分隔符是由Hadoop决定的。

####  定义bucket

```sql
CREATE TABLE page_view(viewTime INT, userid BIGINT,
                page_url STRING, referrer_url STRING,
                ip STRING COMMENT 'IP Address of the User')
COMMENT 'This is the page view table'
PARTITIONED BY(dt STRING, country STRING)
CLUSTERED BY(userid) SORTED BY(viewTime) INTO 32 BUCKETS
ROW FORMAT DELIMITED
        FIELDS TERMINATED BY '1'
        COLLECTION ITEMS TERMINATED BY '2'
        MAP KEYS TERMINATED BY '3'
STORED AS SEQUENCEFILE;
```

`COLLECTION ITEMS TERMINATED BY '2'` 数组分隔符。
`MAP KEYS TERMINATED BY '3'` keys分隔符。


####  导入数据

```sql
CREATE EXTERNAL TABLE page_view_stg(viewTime INT, userid BIGINT,
                page_url STRING, referrer_url STRING,
                ip STRING COMMENT 'IP Address of the User',
                country STRING COMMENT 'country of origination')
COMMENT 'This is the staging page view table'
ROW FORMAT DELIMITED FIELDS TERMINATED BY '44' LINES TERMINATED BY '12'
STORED AS TEXTFILE
LOCATION '/user/data/staging/page_view';

hadoop dfs -put /tmp/pv_2008-06-08.txt /user/data/staging/page_view

FROM page_view_stg pvs
INSERT OVERWRITE TABLE page_view PARTITION(dt='2008-06-08', country='US')
SELECT pvs.viewTime, pvs.userid, pvs.page_url, pvs.referrer_url, null, null, pvs.ip
WHERE pvs.country = 'US';
```

> 根据hdfs文件建立外部表，再从外部表插入到内部表。

如果外部表定义和内部表定义相同则可以直接插入。

```sql
LOAD DATA INPATH '/user/data/pv_2008-06-08_us.txt' INTO TABLE page_view PARTITION(date='2008-06-08', country='US')
```

####  动态分区


插入数据到分区表时，需要手动将不同分区的数据插入到对应的分区中，有几个分区就要写几条INSERT语句。

```sql
FROM page_view_stg pvs
INSERT OVERWRITE TABLE page_view PARTITION(dt='2008-06-08', country='US')
       SELECT pvs.viewTime, pvs.userid, pvs.page_url, pvs.referrer_url, null, null, pvs.ip WHERE pvs.country = 'US'
INSERT OVERWRITE TABLE page_view PARTITION(dt='2008-06-08', country='CA')
       SELECT pvs.viewTime, pvs.userid, pvs.page_url, pvs.referrer_url, null, null, pvs.ip WHERE pvs.country = 'CA'
INSERT OVERWRITE TABLE page_view PARTITION(dt='2008-06-08', country='UK')
       SELECT pvs.viewTime, pvs.userid, pvs.page_url, pvs.referrer_url, null, null, pvs.ip WHERE pvs.country = 'UK';
```

这样做很不方便，应该事前我们可能不知道到底有多少个分区。动态分区就是为了解决这一问题设计的。在插入时，系统自动帮我们解决分区的问题。

```sql
FROM page_view_stg pvs
INSERT OVERWRITE TABLE page_view PARTITION(dt='2008-06-08', country)
       SELECT pvs.viewTime, pvs.userid, pvs.page_url, pvs.referrer_url, null, null, pvs.ip, pvs.country
```

语法区别：

- country列出现在partition参数中，代表动态分区。
- country列加到select语句中。

语义：

- 如果存在分区，则覆盖分区。（INSERT OVERWRITE）
- 因为Hive分区对应HDFS目录，所以分区值必须满足HDFS路径格式。（Java URI）
- 分区值如果不是string类型则转换成string类型。
- 如果分区值为空或者其他非法值，则替换成HIVE_DEFAULT_PARTITION{}。
- 如果操作不当动态分区可能产生大量的分区。为了防止这种情况，可以定义如下参数：
  -  `hive.exec.max.dynamic.partitions.pernode` 单节点创建的最大分区数。
  -  `hive.exec.max.dynamic.partitions` 一条DML语句创建的最大分区数。
  -  `hive.exec.max.created.files` 最大文件数。
- 为了防止用户意外将所有分区都指定为动态分区，设置参数hive.exec.dynamic.partition.mode=strict来保证最少有一个分区是静态分区。Hive 0.9.0之后动态分区是默认开启的。

优化：
- use tez
- ORC file 
 - predicate pushdown only read records satify where clause
 - compression
- use vectorization process records in 1024 rows batch instead of each row
- cost based query optimization ananlyze table before querying
- write good sql avoid joining

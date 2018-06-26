---
layout: post
title: GCP BigQuery BigTable
---

## 降低花费

使用数据预览和费用预估。
- query validator 
- Performing a dry run --dry_run 
- Limit query costs by restricting the number of bytes billed
- LIMIT doesn’t affect cost
- use the default table expiration to automatically delete the data

## 优化

- Avoid SELECT *
- Prune partitioned queries use the _PARTITIONTIME pseudo column to filter the partitions.
- Denormalize data whenever possible Nested and repeated fields can maintain relationships without the performance impact of preserving a relational (normalized) schema. shuffling means data transfer
- Use external data sources appropriately
- Avoid excessive wildcard tables
- Reduce data before using a JOIN
- Avoid tables sharded by date. Sharded tables require BigQuery to maintain schema, metadata, and permissions for each shard. 
- Avoid repeatedly transforming data via SQL queries
- Avoid JavaScript user-defined functions. Use native UDFs instead.
- Use approximate aggregation functions. If your use case supports it, use an approximate aggregation function.
- Order query operations to maximize performance. push complex operations, such as regular expressions and mathematical functions to the end of the query.
- Optimize your join patterns. Start with the largest table.
- Carefully consider materializing large result sets. BigQuery limits cached results to approximately 128MB compressed. using the dataset's default table expiration.
- Avoid self-joins. Use a window function instead.
- If your query processes keys that are heavily skewed to a few values, filter your data as early as possible.
- Unbalanced joins
- DML statements that update or insert single rows. batch update and insert not single. if single, stream it.
- Take advantage of long-term storage. Each partition of a partitioned table is considered separately for long-term storage pricing.
- Use the pricing calculator to estimate storage costs

## BigTable
类似于HBase A Bigtable is a sparse, distributed, persistent multidimensional sorted map. Empty cells in a Cloud Bigtable table do not take up any space. If a row does not include a value for a specific key, the key/value entry is simply not present. Column qualifiers take up space in a row. it is not a good solution for storing less than 1 TB of data. If you need to store highly structured objects in a document database, with support for ACID transactions and SQL-like queries, consider Cloud Datastore.
Production: A standard instance with either 1 or 2 clusters, as well as 3 or more nodes in each cluster. You cannot downgrade a production instance to a development instance.
Development: A low-cost instance for development and testing, with performance limited to the equivalent of a 1-node cluster. There are no monitoring or throughput guarantees; replication is not available; and the SLA does not apply. You can upgrade a development instance to a production instance at any time.
HDD storage is sometimes appropriate for very large data sets (>10 TB) that are not latency-sensitive or are infrequently accessed.
In Cloud Bigtable, reads and writes are always atomic at the row level.
Each Cloud Bigtable zone is managed by a master process, which balances workload and data volume within clusters. The master splits busier/larger tablets in half and merges less-accessed/smaller tablets together, redistributing them between nodes as needed.
Tablets are stored on Colossus, Google's file system, in SSTable format. Immutable.
The average CPU utilization across all nodes in the cluster.
In general, this value should be a maximum of 70%, or 35% if you use replication with multi-cluster routing. This maximum provides headroom for brief spikes in usage. This configuration helps ensure that if an automatic failover is necessary, the healthy cluster has enough capacity to handle all of your traffic.
If a cluster exceeds this maximum value for more than a few minutes, add nodes to the cluster.
CPU utilization of hottest node 90%.
每个节点ssd最大2.5tb，hdd最大8tb，保证不超过70%
SS INDEX在内存中，每次add,delete,modify key操作在新的ss index和内存temp table中进行，先查temp表在查硬盘表。temp table超过数量持久化，生成新的temp table。temp table超过数量再进行合并。
水平扩展，master包括METATABLE，keys：tablename_ending-row-key
机器存储若干tablet

### schema
- Each table has only one index, the row key. There are no secondary indices.
- Rows are sorted lexicographically by row key,
- All operations are atomic at the row level.
- Ideally, both reads and writes should be distributed evenly 
- In general, keep all information for an entity in a single row.
- Related entities should be stored in adjacent rows, which makes reads more efficient.
- Cloud Bigtable tables are sparse. Empty columns don't take up any space. As a result, it often makes sense to create a very large number of columns, even if most columns are empty in most rows.

#### Choosing a row key
- Reverse domain names
- String identifiers
- Timestamps With a reversed timestamp, the records will be ordered from most recent to least recent.
- Multiple values in a single row key
#### avoid
- Sequential numeric IDs
- Static, repeatedly updated identifiers
- Hashed values

Grouping data into column families enables you to retrieve data from a single family, or multiple families, rather than retrieving all of the data in each row. it often makes sense to treat column qualifiers as data. tall and narrow tables are best suited for time-series data. This is for two reasons: Storing one event per row makes it easier to run queries against your data. Storing many events per row makes it more likely that the total row size will exceed the recommended maximum (see "Rows can be big but are not infinite"). As an optimization, you can use short and wide tables, but avoid unbounded numbers of events. 
Prefer rows to column versions. It is acceptable to use versions of a column where the use case is actually amending a value, and the value's history is important.
Choosing a row key that facilitates common queries is of paramount importance to the overall performance of the system.
By default, prefer field promotion. Use salting only where field promotion does not resolve hotspotting. 
In general, keep row sizes below approximately 100 MB. In general, keep column values below approximately 10 MB.
Reverse timestamps only when necessary
Keep related data in the same table, keep unrelated data in different tables
Look for opportunities to denormalize data to satisfy queries, but only include as much data as required by the queries.
Ensure your common queries are as efficient as possible by retrieving data from as few column families as possible.

### replication
An instance's clusters must be in unique zones that are within the same region. Placing clusters in different zones helps ensure that you can access your data even if one Google Cloud Platform zone becomes unavailable.
Isolate serving applications from batch reads
Improve availability
Provide near-real-time backup

#### app profile
An application profile, or app profile, stores settings that tell your Cloud Bigtable instance how to handle incoming requests from an application. When one of your applications connects to a Cloud Bigtable instance, it can specify an app profile, and Cloud Bigtable uses that app profile for any requests that the application sends over that connection.
It also prevents you from enabling single-row transactions in an app profile that uses multi-cluster routing, because there's no safe way to enable both of these features at once.

If your data is in Avro format, which is self-describing, BigQuery can determine the schema directly.
If the data is in JSON or CSV format, BigQuery can auto-detect the schema, but manual verification is recommended.
To automate the process you can set up a Cloud Function to listen to a Cloud Storage event that is associated with arriving new files in a given bucket and launch the BigQuery load job.
There is a charge for streaming data.
All table modifications in BigQuery are ACID compliant.
you can use BigQuery's native support for date-partitioned tables and partition expiration.
You design, develop, and test the upgrade in parallel while the previous version of the data warehouse is serving the analysis workloads.

https://cloud.google.com/solutions/bigquery-data-warehouse
### SCD
- Technique 1: view switching
- Technique 2: in-place partition loading
- Technique 3: update data masking
- SCD Type 1: overwrite attribute value
- SCD Type 2: change attribute value and maintain history
use Apache Airflow API for BigQuery. For simpler orchestrations, you can rely on cron jobs. 
BigQuery does not use or support indexes. UDFs are written in JavaScript and can include external resources, such as encryption or other libraries.

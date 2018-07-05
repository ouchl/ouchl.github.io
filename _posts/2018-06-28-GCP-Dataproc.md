---
layout: post
title: GCP Dataproc
---

- Log processing—With minimal modification, you can process large amounts of text log data per day from several sources using existing MapReduce.

- Reporting—Aggregate data into reports and store the data in BigQuery. Then you can push the aggregate data to applications that power dashboards and conduct analysis.

- On-demand Spark clusters—Quickly launch ad-hoc clusters to analyze data is stored in blob storage using Spark (Spark SQL, PySpark, Spark shell).

- Machine learning—Use the Spark Machine Learning Libraries (MLlib), which are preinstalled on the cluster, to customize and run classification algorithms.

hadoop, spark, spark sql, hive, pig
可以创建多个cluster，解决冲突问题。无法升级现有cluster可以销毁重建。The most cost-effective and flexible way to migrate your Hadoop system to GCP is to shift away from thinking in terms of large, multi-purpose, persistent clusters and instead think about small, short-lived clusters that are designed to run specific jobs.
tailor cluster configurations for individual jobs

sqoop hdfs to relational db
oozie = workflow dag
stack driver resource type and metrics

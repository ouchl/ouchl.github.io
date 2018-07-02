---
layout: post
title: GCP BigTable
---
一个node最多2559GB.

Tall schemas are often used for storing time-series events, data that is keyed in some portion by a timestamp, with relatively fewer columns per row. Wide schemas follow the opposite approach, a simplistic identifier as the row key along with a large number of columns.

Real-time application data—Cloud Bigtable can be accessed from applications running in App Engine Flexible, Kubernetes Engine, and Compute Engine for real-time live-serving workloads.

Stream processing—As data is ingested by Cloud Pub/Sub, Cloud Dataflow can be used to transform and load the data into Cloud Bigtable.

IoT time series data—Data captured by sensors and streamed into Cloud Platform can be stored using time-series schemas in Cloud Bigtable.

Adtech workloads—Cloud Bigtable can be used to store and track ad impressions, as well as a source for follow-on processing and analysis using Cloud Dataproc and Cloud Dataflow.

Data ingestion—Cloud Dataflow or Cloud Dataproc can be used to transform and load data from Cloud Storage into Cloud Bigtable.

Analytical workloads—Cloud Dataflow can be used to perform complex aggregations directly from data stored in Cloud Bigtable, and Cloud Dataproc can be used to execute Hadoop or Spark processing and machine-learning tasks.

Apache HBase replacement—Cloud Bigtable can also be used as a drop-in replacement for systems built using Apache HBase, an open source database based on the original Bigtable paper authored by Google. Cloud Bigtable is compliant with the HBase 1.x APIs so it can be easily integrated into many existing big-data systems. Apache Cassandra utilizes a data model based on the one found in the Bigtable paper, meaning Cloud Bigtable can also support several workloads that leverage a wide-column-oriented schema and structure.

While Cloud Bigtable is considered an OLTP system, it does not support multi-row transactions, SQL queries or joins. For those use cases, consider either Cloud SQL or Cloud Datastore.

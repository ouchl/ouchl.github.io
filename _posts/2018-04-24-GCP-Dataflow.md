---
layout: post
title: GCP Dataflow
---

## stream vs batch
stream实时，连续，低延迟。batch割裂数据，处理滞后。数据完整性，延迟，费用。event time processing time have skew time watermark older than it drop 
out of order data. watermark太小，aggregate等待时间太长，watermark太大，结果不准确。fixed window sliding window sessoins window

## dataflow vs dataproc
- 存在Hadoop环境依赖，使用dataproc
- 有devops dataproc serverless dataflow
流处理 dataflow
批处理 都可以
迭代处理和笔记本 dataproc
使用 Spark ML 进行机器学习 dataproc
为机器学习进行预处理	dataflow

## 优点
- autoscaling
- 更低的运营开销
- 以统一的方式开发批处理流水线和流处理流水线
- 使用 Apache Beam
- 支持跨 Cloud Dataflow、Apache Spark 和 Apache Flink 移植运行中的流水线

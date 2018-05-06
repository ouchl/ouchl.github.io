---
layout: post
title: GCP Overview
---

## 概念

region -> zone
global resources, regional resources, zonal resources

project定义名字域

三种管理方式
- Google Cloud Platform Console web界面
- Command-line interface cloud shell页面命令行
- Client libraries 用途获取服务和管理资源

## 计算服务

- serverless environment
- managed application platform
- container technologies
- cloud-based infrastructure
灵活性递增

### Serverless computing
JS写的回调函数。类似于浏览器上的事件。不同的回调函数可以同时运行，但是同一个回调函数只能运行一个实例。无状态函数，不依靠之前函数的执行。细粒度适用于轻量级api和webhook。
#### HTTP函数

通过http执行函数。ExpressJS Request和Response对象。最多执行一次。需要返回错误代码。

#### background函数

参数event和callback。callback处理函数运行结果，第一个参数表示错误。没有callback参数可以返回一个promise或者值。最少调用一次，因为异步没有调用者，可以失败时重试。
需要返回错误对象（callback）或者rejected promise。

#### 执行环境
##### 冷启动
部署函数和替换现有实例。启动一个新的函数实例包括加载nodejs和运行代码。
##### 函数实例生命周期
弹性周期，反复使用除非实例数量被限制或者函数崩溃。所以可以把一些可以缓存的数据放到函数的全局变量里，加速执行速度。

#### 事件和触发器

触发器是对感兴趣的一个或者一组事件的声明。把函数绑定到触发器上可以捕获事件。不能把一个函数绑定到多个触发器上。

使用场景
- ETL 监控Cloud Storag上文件变化
- webhook监控http请求
- 轻量api
- 移动后端函数

### Application platform

- 标准环境  Python 2.7, Java 8, Java 7, PHP 5.5, and Go 1.8, 1.6.
- 灵活环境  Python 2.7/3.6, Java 8, Go 1.8, Node.js, PHP 5.6, 7, .NET, and Ruby. 自定义可以使用任何语言
- 谷歌负责托管，scaling，监控
- 使用app engine sdk模拟gcp环境进行开发
- Google Cloud SQL：MySQL or PostgreSQL。App Engine Datastore：NoSQL datastore。Google Cloud Storage存储大文件。
- Cloud Endpoints实现应用程序间数据传输的标准环境。
- 使用内置的管理服务如邮件和用户管理。
- 使用Cloud Security Scanner
- 使用App Engine launcher GUI application或者命令行来部署
- 标准环境只支持Central US or Western Europe

### Containers容器

#### Kubernetes Engine分布式部署
- 集群中每个节点安装docker和Kubelet agent
- 使用JSON文件定义docker容易的要求
- Google Container Registry存储docker镜像。可以使用http上传或下载镜像文件。
- 一组容器组成一个pod。pod内的容器共享资源，如网络连接等。
- 创建和管理复制pod控制器。
- 创建和管理服务。前端客户和后端pod解耦。
- 创建外部网络负载均衡器。

### Virtual machines虚拟机

gcp无管理的计算服务是Google Compute Engine。

- 使用虚拟机（实例）运行应用程序。
- 选择region和zone。
- 选择软件环境。
- 从镜像创建实例。
- 使用gcp存储技术或者第三方技术。
- 使用Google Cloud Launcher部署预配置软件包如LAMP。
- 使用实例组管理多个实例。
- autoscaling with an instance group当负载高时添加实例，负载低时减小实例。
- 添加减小硬盘数量。
- 使用ssh连接。

### 结合不同选项

可以结合App Engine和Compute Engine利用各自优点。

## 存储服务

- Cloud SQL
- Cloud Spanner关系型数据库，水平可扩展，提供事务一致性。
- NoSQL：Cloud Datastore（支持事务，高可用，贵，支持二级索引，持久化的hashmap），Cloud Bigtable（大容量，便宜，HBase API，只有一个索引Row Index）
- Cloud Storage持久化存储，速度慢
  - Multi-Regional提供高可用和高冗余
  - Regional提供高可用和本地化存储。
  - 归档存储Nearline适合访问频率不到每月一次
  - 归档存储Coldline适合备份，不到一年一次
- Compute Engine持久化存储。

[选择使用的服务类型](https://cloud.google.com/storage-options/)

## 网络服务

每个虚拟机实例只能附加到一个网络上。每个项目有一个默认网络。可以创建附加网络，但是不同项目之间的网络不能共享。

### 负载均衡

- 网络负载均衡可以根据IP数据包在一个region内分发流量。包括地址端口协议。
- HTTP/HTTPS负载均衡。可以保证流量转发到最近的region，如果有其他限制如不可用，带宽到达上线则转发到第二近的region。还可以根据content type分发流量。

### cloud dns

### 高级连接

已有网络连接到gcp资源
- Carrier Interconnect
- peering connection
- cloud vpn

## 大数据服务

### 数据分析
BigQuery
- 创建自定义scheme
- 从各种源加载数据
- 使用类SQL语言快速查询
- 使用页面ui，命令行，api
- 使用job加载，导出，查询，复制数据
- 权限管理

### 批量和流数据处理
Cloud Dataflow适合大容量计算，可并行处理，ETL

### 异步消息
Cloud Pub/Sub是异步消息服务。你的应用程序可以把消息发送到消息发布单元topic。因为cloud pub/sub是全局资源，所以其他应用程序可以订阅这个topic来接收消息。

## 机器学习服务

### ML API
- Google Cloud Video Intelligence API 
- Google Cloud Speech API
- Google Cloud Vision API
- Google Natural Language API
- Google Cloud Translation API
- Dialogflow Enterprise Edition

### Cloud ML Engine

Cloud Machine Learning Engine结合gcp managed infrastructure和TensorFlow

## 开发环境

### Cloud SDK
需要Python 2.7.x.

### Cloud Shell
在浏览器中使用，运行在临时实例中。

### Android Studio

### IntelliJ IDEA

### Cloud Tools for Visual Studio

### Cloud Tools for PowerShell

### Cloud Tools for Eclipse

### Cloud Source Repositories

每个项目都有一个Git仓库。

## 自动部署

### Cloud Launcher
Compute Engine上安装预定义的软件包和配置。

### Cloud Deployment Manager
使用Deployment Manager部署自定义的系统配置。

## cloudproc

hadoop cluster一次job收费。存储使用cloud data storage。



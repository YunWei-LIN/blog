---
title: hadoopLive
date: 2017-03-06 19:05:16
tags: [bigData, hadoop, live]
categories: hadoop
---


主要内容
* 安装 Hive
* 概念
* Hive 脚本
* 整合 MapReduce

Hive 就是 Hadoop 平台的开源数据仓库， HiveQL 就是基于 HDFS 系统的 SQL 引擎， Hive 转换成多阶段的 MapReduce 任务。

......

<!-- more -->


## 安装
+ 下载
+ `HIVE_HOME` 环境变量
+ `$HIVE_HOME/bin` 加入 PATH 环境变量

在Hadoop集群中使用Hive查询， 前提
+ Hadoop 必须存在于 PATH 环境变量中， 或者 `HADOOP_HOME` 环境变量已经设置为 Hadoop 的安装目录
+ 在 HDFS 中创建 `/tmp` 和 `/user/hive/warehose` 目录， 权限为 `g+w`
+ 为多个用户配置元数据存储


## 概念
+ 数据库
+ 表
默认 托管表
    - 托管表
    - 扩展表 external
    方便访问或修改 HDFS 已经存在的结构化数据。
+ 视图
+ 分区
表分区或切片， 最好情况下， 分区的划分条件总是能够对应 where 语句的部分查询条件。
分区使用 HDFS 的子目录功能实现。不支持大量的子目录。
  
+ 桶 bucket
分区中的数据可以被进一步拆分成桶。在分区数量过于庞大以至于可能导致文件系统崩溃时， 建议使用桶。
桶的数据是固定的， Hive使用基于列的哈希函数对数据打散， 并分发到各个不同的桶中从而完成数据的分桶过程。
桶也用来实现高效的Map端连接操作。
  
+ 索引
  
+ 序列化和反序列化

## 编译细节
使用 `EXPLAIN` 语句。

## Hive 脚本
Hive查询可以在脚本中顺序执行，可以使用参数。
例如
```sql
load data loal inpath '${hiveconf:src}' overwrite into table videos;
  
hive -hiveconf src=data/videos.txt -f hiveql2.txt
```

## 整合 MapReduce
大规模应用中更常见的是同时使用 MapReduce 和 Hive。 MapReduce 可以读取或写入 Hive 扩展表或 Hive 能够访问的文件。
在典型数据分析流程中，前端的MapReduce 接收非结构化或不规则的数据， 并转化成 Hive 可以使用的结构化数据。

+ 读取 Hive 扩展表
Hive 在 HDFS 中表现为文件路径。Mapper的InputFormat必须要和 Hive 表的存储格式匹配。 如果使用文本文件存储， 那么可以使用TextInputFormat。 
如果使用2进制序列文件存储， 那么应该使用 SequenceInputFormat。
+ 写入 Hive 扩展表
MapReduce 任务的输入目录设置为 Hive 扩展表在 HDFS 中的位置即可。扩展表的文件格式必须和MapReduce任务相匹配。
<span style="color:red"> 特： Hive 默认使用 \0001 (CTRL+A) 作为列分隔符。</span> 

## 创建分区

## 用户定义函数


























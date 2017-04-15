---
title: hadoop MapReduce
date: 2017-03-06 17:19:43
tags: [bigData, hadoop, 全排序]
categories: hadoop
---


主要内容
hadoop MapReduce 全排序 日志 监控



......

<!-- more -->

## hadoop I/O 

### 键值特征
+ 键和值 都可以序列化和反序列化
+ Reduce的键支持用户自定义排序

### Writable 和 WritableComparable接口
+ Writable接口方法
    - write
        序列化， 将实例中所有原始属性写道java.io.DataOutput类型的输出流。
    - readField
        反序列化，利用从java.io.DataInput类型的输入流中抓取的数据重新创建Writable实例。
  
    字段顺序必须和 `write()` 和 `readField()` 方法中的顺序保持一致。
  
+ 优秀实践
<span style="color:red">Mapper 输出的键类</span>或 <span style="color:red">Reduce 输入的键类</span>都应该实现 WritableComparable接口， 
其他所有类型（例如 Mapper`输入`使用的键和值类，Mapper`输出`的值类， Reduce`输出`的键类）


## 全排序
1. reduce键的排序
键到reduce之前排序， 在WritableConparable键中实现排序逻辑或自定义Comparator

2. reduce键的转换为索引数值

3. 自定义Partitioner
 1. 清楚reduce键的索引范围
 2. 根据reduce键的索引范围 和 reduce的数目 实现划分逻辑

## 二级分类/二次排序
### 问题描述：
+ 根据键类的某些属性聚拢记录，确保根据键类的某些属性分配到相同的reduce，确保在一次reduce调用中被处理， 这些记录的键只有部分相等。
+ 根据键类的某些属性排序

### 重要组件
+ 自定义的WritableComparable键类
用于Mapper的输入和Reduce的输入
+ 自定义Partitioner类
用来确保根据键类的某些属性分配到相同的reduce
+ Sorting Comparator
使用自定义的WritableComparable键类对记录进行完全排序。
+ Grouping Comparator
在Reduce端对键进行聚拢，确定Grouping标准，Grouping标准决定从Mapper发往Reduce的哪些值将在一次reduce调用中被使用

## 连接Join
SQL的常用功能：连接（join），hadoop有2中方法实现：
+ Mapper端连接
当数据集中的一个非常小的时候使用。
 
+ Reduce端连接
当数据集都非常大并且其中一个无法完全在内存中缓存时使用，当连接的标准是结果的数量对于任何给定的连接数据集实例都是非常小的。
连接的键作为Mapper的输出键， Mapper输出值还包含一个标识属性来指示这个记录（值记录）是来自哪一个记录集。


## 日志
### Web界面
* Job Manager Web 界面
http://job_history_manager_host:19888
  
* 资源管理器
http://resource_manager_host:8088

### 命令行界面
能与命令行交互的日志文件是 $HADOOP_YARN_HOME/bin/yarn下的日志

获得整个作业日志
```sh
$HADOOP_YARN_HOME/bin/yarn logs -applicationId <application i>
```

获得某个容器的日志
```sh
$HADOOP_YARN_HOME/bin/yarn logs -applicationId <application i> -containerId <container id> -nodeAddress <nodemanager ip_add:prot>
```

获得其他用户的作业日志
```sh
$HADOOP_YARN_HOME/bin/yarn logs -applicationId <application i> -appOwner <user id> -containerId <container id> -nodeAddress <nodemanager ip_add:prot>
```

### 保存
`yarn.log-aggregation.retain-seconds` : 指定多少秒后删除日志， 负数标识禁用此功能并永久保存。默认值是-1。
`yarn.log-aggregation.retain-check-interval-seconds` : 决定日志保存检测的频率， 默认为-1, 如果该值是0或负数， 默认是yarn.log-aggregation.retain-seconds 的十分之一。
`yarn.nodemanager.log.retain-seconds` : 当日志是非聚集日志时， 日志在本地目录的保存时间， 默认10800秒， 3小时。


## 监控
开源 Ambari

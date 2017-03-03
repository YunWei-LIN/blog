---
title: hadoop 全排序
date: 2017-02-28 17:19:43
tags: [bigData, hadoop, 全排序]
categories: hadoop
---


主要内容
* 步骤



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

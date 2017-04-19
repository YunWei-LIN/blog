title: 【mysql 管理之道】读书笔记三
date: 2017-4-19 18:53:57
tags: mysql
categories: mysql
---

概要

MySQL5.7新特性
故障处理
性能调优
备份和恢复
高可用架构

拜读 贺春旸先生 的 【mysql 管理之道】 有感， 特此梳理和记录。
本篇为第三篇， 主要包括高可用，读写分离， 分库分表

<!-- more -->
## 高可用架构
### MHA 

MHA （master high availability） : 会把丢失的数据在每个slave节点上补齐。
[MHA github](https://github.com/yoshinorim/mha4mysql-manager)
提供3种故障转移模式：

+ master 自动监控和故障转移
需要开启半自动复制（semi replication）
+ 手工处理master故障转移
机器长时间启动不了或者崩溃恢复时间太长，并可能丢失数据。
+ 在线平滑切换
如果机器需要维护， 将master在线切换到其他主机上

page 270

### 读写分离
读/写分离 技术为 一个 master数据库， 多个 slave 数据库。 master 负责数据更新和实时数据查询， slave 负责非实时数据查询。实际应用， 大部分是读多写少， 而读数据通常耗时时间比较长， cpu内存占用都比较多， 对此通常就把查询从主库分离，采用多个salve， 负载均衡， 减轻每个从库的眼里。

#### 目标
既有效减轻master库的压力， 又可以把用户查询数据的请求分发到不通slave库

#### 基本原理
master 处理增， 删， 改操作， slave 处理select 操作， replication 负责把数据变更同步到集群的 slave库。

#### 实现方式
+ MaxScale
[MaxScale 官网](https://mariadb.com/blog-tags/maxscale)
[MaxScale github](https://github.com/mariadb-corporation/MaxScale)

+ OneProxy

+ Atlas

+ MyCAT

### 分库分表 
+ OneProxy

+ Atlas

+ MyCAT

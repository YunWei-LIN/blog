title: 【mysql 管理之道】读书笔记一
date: 2017-4-15 12:53:57
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
本篇为第一篇， 主要包括MySQL5.7新特性

<!-- more -->

## MySQL5.7新特性

### 安全性提高
* SSL
默认可开启SSL连接
  
* sql_mode
默认启用 `STRICT_TRANS_TABLES` 严格模式， 用于进行数据的严格校验，错误数据不能插入，报error， 并且事务回滚。

### InnoDB 存储引擎提升
* 更改索引名不锁表

* 在线DDL修改varchar字段属性不锁表
	这里还是有坑的，一定要执行 `show processlist` 命令并观察， 是否有慢SQL正在操作，如有则不要进行 alter table

* InnoDB 支持中文全文索引
`ngram`解析器， `ngram_token_size` 参数不可动态修改， 停止词不能动态修改， 需要重建一次全文索引

* InnoDB Buffer Pool 预热
数据库重启时， 将之前频繁访问的数据重新加载回Buffer中， 快速恢复之前的性能状态。
只需要在 `my.cnf` 里加入
	```sh
	innodb_buffer_pool_dump_at_shutdown = 1  # 关闭时把热数据备份到本地， 正常关闭或pkill时才起作用
	innodb_buffer_pool_load_at_startup = 1   # 启动时把热数据加载到内存
	```
  
* 可在线调整 Innodb_buffer_pool_size 而不用重启 MySQL

* 回收 undo log 回滚日志物理文件空间
undo log 回滚日志 在 MySQL 5.6 之前， 保存在共享表空间ibdata1文件里的， MySQL 5.6 支持 单独表空间， MySQL 5.7 支持 在线收缩
初始化数据库前 在 `my.cnf` 指定如下参数
```sh
innodb_undo_directory = /data2/ # 
innodb_undo_log_truncate = 1 # 开始在线回收， 支持动态设置
innodb_undo_logs = 128 # undo 回滚段的数量， 至少大于等于 35， 默认 128
innodb_undo_tablespaces = 4 # 指定有多少个 undo log 文件， 必须大于 2
innodb_max_undo_log_size = 1G # 可省略，默认1G， 当超过这个阈值，会触发 truncate 回收动作， truncate后 空间缩小到 10MB
innodb_purge_rseg_truncate_frequncy : 控制回收 undo log 的频率
```

* 通用表空间
可以理解为加强版独立表空间，可以指定某些表存放在同一个 share.ibd 文件里。需要开始参数 `innodb_file_per_table = 1`

* 修改 InnoDB redo log 事务日志文件大小 人性化
	- 执行 set global innodb_fast_shutdown = 0; 将所有脏页刷到磁盘
	- 执行 mysqladmin shutdown 关闭数据库
	- 在 my.cnf 修改 innodb_log_file_size 参数值
	- 执行 mysqld_safe --default-file=/etc/my.cnf --user=mysql& , 最后启动 MySQL
在 5.5 版本，启动mysql之前， 必须 mv ib_logfile* /bak, 将redo log 移动到 bak 目录

* 死锁可 打印到 错误日志
命令 show engine innodb status；
在 `my.cnf` 中 设置 `innodb_print_all_deadlocks = 1`

* 支持 InnoDB 只读事务

### JSON 格式支持
MySQL 5.7 版本支持 原生的 JSON 格式，将关系型数据库和文档型NoSQL数据库集成一身。
[JSON TYPE](https://dev.mysql.com/doc/refman/5.7/en/json.html)
[JSON FUNCTION](https://dev.mysql.com/doc/refman/5.7/en/json-function-reference.html)

### 函数索引
虚拟列方式支持函数索引

### 功能提升
* 一张表多个 insert/delete/update  触发器

* 支持 explain update

* Ctrl + C 组合键不会退出客户端

### 优化器改进
* 子查询select 采用半连接优化
* 优化排序 limit
* 优化IN条件
* 支持 Multi Range Read 优化
	大表进行范围扫描，通过 MRR 优化减少随机I/O
	可以通过 `show variables like 'optimizer_switch'; ` 来查看是否开启 MRR。

### 半同步复制改进
复制功能默认是异步的， 异步复制可以提供最佳性能， 主库把 Binlog 日志发给从库， 这一动作就完成了， 并不验证从库是否接收完毕。
从库可能没有接收，从而导致复制数据丢失。
半同步复制模式，可以确保从服务器接收完主服务器发送的 Binlog 日志文件并写入自己的中继日志（relay log）里， 然后给主服务器一个反馈， 这是主服务器才结束操作。
需要主从服务端都开启半同步模式。

### GTID 全局事务标识符
由uuid和事务id 组成。 主从切换后，同步失败，MySQL 可以通过GTID 自动找点同步。 需要主从都开启。

### salve 支持多源复制
多主一从

### 同步复制过滤不用重启 mysql 服务进程

title: 【mysql 管理之道】读书笔记二
date: 2017-4-16 12:53:57
tags: mysql
categories: mysql
---

概要

MySQL5.7新特性
故障处理
性能调优
备份和恢复
高可用架构

拜读 贺春旸先生 的 【mysql 管理之道】 有感， 特此梳理和记录。 主要适合 MySQL 5.6， MySQL 5.7.
本篇为第二篇， 主要包括故障诊断，性能调优， 备份和恢复。

<!-- more -->
## 故障诊断
### dstat
Linux 性能监控工具 dstat

### 故障与处理
#### 连接数过多导致程序连接报错的原因
keys：
* max_connections 通常 500 ~ 1000 比较合理， 一直增大不能治本
* wait_timeout (服务器关闭非交互连接之前等待活动的秒数) ， 一般设置为 100 就可， 默认是 28800 秒
* 重启数据库导致早高峰连接数过多
	可使用 InnoDB Buffer Pool 预热
* 磁盘高负荷拖垮MySQL
	加内存， InnoDB_Buffer_Pool 最大可调整成内存的 70%

#### binlog_format
binlog_format 有3种格式 ， statement， row 和 mixed
- statement 记录的是实际的SQL语句

- row 记录的是实际行的变更

- mixed 默认是 statement， 在如下情况会转换为 row。
	NDB引擎，表的DML； SQL语句包含UUID()函数； 自增字段被更新； 包含 INSERT DELAYED 语句； 使用用户定义函数 UDF； 使用临时表。

row格式 安全稳定，binlog 可能会很大， 性能较差；
statement格式 性能较好，可能会导致同步数据问题

#### 未设置 swap 分区导致内存耗尽，死机
swappiness 设置为 0， 只能减少使用 swap 的概率， 不能避免 Linux 使用 swap，所以内存被耗尽后， 主机仍旧会被Hang死。
增加 swap
监控内存， 达到一定量， 重启 MySQL

#### 恢复 人工误删除 InnoDB ibdata 数据文件
切记 千万千万不要杀死 mysql 进程！！！！！
page 130 ~ 132

#### 闪回功能
** Binlog 必须是 row 格式！！！ **

* 恢复update忘记where条件误操作
page 133

* 恢复delete忘记where条件误操作
page 141


### 同步复制报错故障处理
page 145

#### 干净清除 slave 同步信息
```sql
reset slave all; -- reset slave 会把 master.info 和 relay-log.info 文件删除， 但同步信息还在。
show slave status;
```


## 性能调优
### 三范式
设计数据库需要最大可能遵守三范式； 有时为了性能，我们需要有意违反三范式，适度做冗余，提高查询效率。注意，反范式是适度的，必须有充足的理由。

### 字段类型
保小不保大，能用占用字节少的字段就不用大字段。

#### 数值类型

|类型|字节|最小值|最大值|
| -- |:--:| -- |
|tinyint| 1 |有符号 -128  <br> 无符号 0| 有符号 127  <br> 无符号 255|
|smallint| 2 |有符号 -32 768  <br> 无符号 0| 有符号 32 767  <br> 无符号 65 535|
|mediumint| 3 |有符号 -8 388 608  <br> 无符号 0| 有符号 8 388 607  <br> 无符号 16 777 215|
|int| 4 |有符号 -2 147 483 648  <br> 无符号 0| 有符号 2 147 483 647  <br> 无符号 4 294 967 295|
|bigint| 8 |有符号 -9 223 372 036 854 775 808  <br> 无符号 0| 有符号 9 223 372 036 854 775 807  <br> 无符号 18 446 744 073 709 551 615|

+ 手机号码可以用 bigint
字符集 utf8，utf8 占用3字节， char（11) 就是 11 * 3 = 33 字节； bigint(20) 占用8字节

+ ip地址可采用 无符号int
mysql 内置函数： inet_aton(), 它负责ip地址转换成数字； inet_ntoa()负责把数字转换为ip地址

+ 根据需求选择最小整数类型

#### 字符类型
char  varchar
char(N) 长度最大255， 超长会被截取， 比N 小会被空格填补； 
varchar(N) 最大长度 65 535， 额外使用 1~2 个字节来存储值得长度， 如果N小于或等于255， 则占用1字节， 否则就是2字节；
char和varchar跟字符编码也有关， latin1占用1个字节， gbk占用2个字节， utf8占用3个字节。

计算varchar(N)的最大长度
+ GBK字符集
（65 535 -2 ） /2 = 32766
+ UTF8字符集
（65 535 -2 ） /3 = 218 44
+ 一行总长度不能超过 65 535

#### 时间类型
|数据类型|值|存储字节|
|--|--|--|
|date|‘0000-00-00’|3字节|
|time|‘00:00:00’|3字节|
|datetime|‘0000-00-00 00:00:00’|8字节|
|timestamp|‘0000-00-00 00:00:00’|4字节|
|year|0000|1字节|

+ timestamp 默认值 current_timestamp, 自动插入，更新时间
+ MySQL 5.5之前， 一个表只允许一个字段timestamp， 自动插入，更新时间
	MySQL 5.6之后， datetime 有了timestamp功能， 自动插入，更新时间， 也可由多个自动插入，更新时间的字段



### 事务隔离
|隔离级别|读数据一致性|脏读|不可重复读|幻读|
|--|--|--|--|--|
|未提交读 read uncommitted|最低级别， 只能保证不读取物理上损坏的数据|是|是|是|
|已提交读 read committed|语句级|否|是|是|
|可重复读 repeatable read|事务级|否|否|是|
|可序列化 serializable|最高级别，事务级|否|否|否|

oracle 和 SQL server 的默认隔离级别 是 read committed； MySQL 的默认隔离级别是 Repeatable read。


### my.cnf 配置文件调优
page231
per_thread_buffers + global_buffer 不能大于物理内存

#### per_thread_buffers 
可以理解为 oracle 的 PGA， 为每个连接到 MySQL 的用户分配的内存。
per_thread_buffers = (read_buffer_size + read_rnd_buffer_size + sort_buffer_size + thread_stack + join_buffer_size + binlog_cache_size) * max_connections

+ read_buffer_size
用于表的顺序扫描，表示每个线程分配的缓冲区大小。默认128KB， 一般在128 ～ 256KB
+ read_rnd_buffer_size
用于表的随机读取，表示每个线程分配的缓冲区大小。64位默认256KB， 一般在128 ～ 256KB
+ sort_buffer_size
在表进行 order by 和 group by 排序操作时， 由于排序没有索引， 会出现 Using filesort， 可用此参数增加每个线程分配的缓冲区大小， 64位默认256KB，一般在128 ～ 256KB， 如果出现 filesort，可增加索引。
+ thread_stack
表示每个线程的堆栈大小。64位默认256KB
+ join_buffer_size
表进行join时，如果没有索引，会出现 Using join buffer， 可用此参数增加每个线程分配的缓冲区大小，64位默认256KB，一般在128 ～ 256KB， 如果出现 Using join buffer，可增加索引。
+ binlog_cache_size
一般来说， 如果数据库没有特别大事务， 写入也不是特频繁，将其设置为1 ～ 2MB, 默认32KB
+ max_connections
该参数设置最大连接数， 默认为100, 一般设置为 500 ～ 1000.

#### global_buffer
可以理解为 oracle 的 SGA， 用于在内存中缓存从数据文件检索出来的数据块。
global_buffer= innodb_buffer_pool_size + innodb_additional_mem_pool_size + innodb_log_buffer_size + key_buffer_size + query_cahce_size

+ innodb_buffer_pool_size
InnoDB 存储引擎的核心数据， 默认128MB， 建议设置成物理内存的 60% ~ 70%

+ innodb_additional_mem_pool_size
设置存储数据字典信息和其他内部数据结构。表越多， 需要这里分配的内存越多， 单位是 byte，参数默认值为8M。如果 InnoDB 用完了内存池中的内存，就会从操作系统中分配内存，同时在 error log 中打入报警信息。
从 MySQL 5.7.4 中移除， InnoDB 实现的内存分配器相比操作系统的内存分配器并没有明显优势，所以在之后的版本，会移除 innodb_additional_mem_pool_size 和 innodb_use_sys_malloc 两个参数，统一使用操作系统的内存分配器。

+ innodb_log_buffer_size
事务日志所使用缓冲区。默认16M， 一般设置16M ～ 64MB

+ key_buffer_size
MyISAM 存储引擎的索引参数

+ query_cahce_size
缓存select 语句和结果集大小的参数。
如果写操作很少，读操作频繁， 那么打开 query_cahce_type=1, 并设置 query_cahce_size 16M， 看情况调试。
反之写操作很频繁，读操作少， 那么打开 query_cahce_type=0, query_cahce_size=0

## 备份和恢复
### 冷备份
#### 备份
+ 关闭mysql
+ 复制 数据目录（包括 ibdata1） 和 日志目录 （包括 ib_logfile0, ib_logfile1, ib_logfile2)

#### 恢复
+ 拷贝上步的数据目录和日志目录
+ 启动mysql

### 逻辑备份
mysqldump
mydumper 

全量，增量备份脚本 page262


### 热备份
xtrabackup - 相对完美的免费开源数据备份工具
page 263





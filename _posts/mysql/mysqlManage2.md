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
本篇主要包括故障诊断，性能调优， 备份和恢复。

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


## 备份和恢复






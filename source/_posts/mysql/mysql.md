title: mysql
date: 2016-11-23 22:53:57
tags: mysql
categories: mysql
---

概要

安装
配置

<!-- more -->

## 安装
centos 安装 最新版 mysql， 目前mysql 是 5.7.16

### yum源
`http://dev.mysql.com/downloads/repo/yum/`下载对应版本的RPM

安装生成yum源配置文件
		sudo rpm -Uvh platform-and-version-specific-package-name.rpm
		yum makecache

### 安装
```
yum install -y  mysql-community-client.x86_64 mysql-community-devel.x86_64 mysql-community-server.x86_64   #基本安装3个组件
```

## 配置
### 初始化
[官方文档](http://dev.mysql.com/doc/refman/5.7/en/data-directory-initialization-mysqld.html)
为安全和备份方便，把mysql的`datadir`配置在非系统目录。

```
# 1 mysql basedir
which mysql
/usr/bin/mysql

# 2 初始化数据文件
cd /usr/bin/
mysqld --initialize --user=mysql --datadir=/home/data//mysql/data

```

### 配置
默认配置文件 `/etc/my.cnf`
```

[mysqld]
#
# Remove leading # and set to the amount of RAM for the most important data
# cache in MySQL. Start at 70% of total RAM for dedicated server, else 10%.
# innodb_buffer_pool_size = 128M
#
# Remove leading # to turn on a very important data integrity option: logging
# changes to the binary log between backups.
# log_bin
#
# Remove leading # to set options mainly useful for reporting servers.
# The server defaults are faster for transactions and fast SELECTs.
# Adjust sizes as needed, experiment to find the optimal values.
# join_buffer_size = 128M
# sort_buffer_size = 2M
# read_rnd_buffer_size = 2M
datadir=/data/mysql/data
socket=/var/lib/mysql/mysql.sock

# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0

log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid
lower_case_table_names=1 # 0：区分大小写，1：不区分大小写

default-storage-engine=INNODB

slow_query_log=on
slow_query_log_file=/data/mysql/log/slow_query_log.log
long_query_time=2

# character-set
# ignore character set information sent by the client
character-set-client-handshake = FALSE
character-set-server = utf8mb4
init_connect='SET NAMES utf8mb4'


# max_allowed_packet=512M
max_connections        = 1000
server_id=1
innodb_buffer_pool_size = 2048M
# innodb_log_file_size=2048M



# mysql model
sql_mode=STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION

[client]
default-character-set = utf8mb4
[mysql]
default-character-set = utf8mb4
```

### 启动
```
# 1 启动
systemctl start mysqld

# 2 开机启动
systemctl enable mysqld
```

### root密码
```
# 1 root 初始密码
╰─# less /var/log/mysqld.log |grep password
2016-11-23T14:10:00.473337Z 1 [Note] A temporary password is generated for root@localhost: G80K__o3eugw

# 2 修改密码
╰─# mysql -uroot -p
Enter password: #1获得的密码
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'new_password';
```

## 有趣的链接
[淘宝MySQL](http://mysql.taobao.org)

[淘宝内部分享：怎么跳出MySQL的10个大坑](http://www.csdn.net/article/2015-01-16/2823591)

title: mysql
date: 2016-11-23 22:53:57
tags: mysql
categories: mysql
---

概要

+ 安装
+ 配置


*更新历史*
2019-7-16 update mysql version 8


<!-- more -->

## 安装
centos 安装 最新版 mysql， 目前mysql 是 8.0

### yum源
`http://dev.mysql.com/downloads/repo/yum/`下载对应版本的RPM

安装生成yum源配置文件
```sh
sudo rpm -Uvh platform-and-version-specific-package-name.rpm
yum makecache
```

### 安装
yum安装 参照 [quick doc](https://dev.mysql.com/doc/mysql-yum-repo-quick-guide/en/)

```
yum install mysql-community-server   #基本安装3个组件
```

## 配置
### 初始化
[官方文档](https://dev.mysql.com/doc/refman/8.0/en/data-directory-initialization.html)
为安全和备份方便，把mysql的`datadir`配置在非系统目录。

```
# 1 mysql basedir
which mysql
/usr/bin/mysql

# 2 init directory
mkdir -p /data/mysql/data
chown -R mysql:mysql /data/mysql

# 3 初始化数据文件
cd /usr/bin/
mysqld --initialize --user=mysql --lower-case-table-names=1 --datadir=/data/mysql/data 

```
生产环境部署,建议不区分大小写, 默认是 `lower-case-table-names=0`
lower-case-table-names=1(不区分) , 0(区分)


### 配置
默认配置文件 `/etc/my.cnf`
```
# For advice on how to change settings please see
# http://dev.mysql.com/doc/refman/8.0/en/server-configuration-defaults.html

[mysqld]
#
# Remove leading # and set to the amount of RAM for the most important data
# cache in MySQL. Start at 70% of total RAM for dedicated server, else 10%.
# innodb_buffer_pool_size = 128M
#
# Remove the leading "# " to disable binary logging
# Binary logging captures changes between backups and is enabled by
# default. It's default setting is log_bin=binlog
# disable_log_bin
#
# Remove leading # to set options mainly useful for reporting servers.
# The server defaults are faster for transactions and fast SELECTs.
# Adjust sizes as needed, experiment to find the optimal values.
# join_buffer_size = 128M
# sort_buffer_size = 2M
# read_rnd_buffer_size = 2M
#
# Remove leading # to revert to previous value for default_authentication_plugin,
# this will increase compatibility with older clients. For background, see:
# https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_default_authentication_plugin
# default-authentication-plugin=mysql_native_password

datadir=/data/mysql/data
socket=/var/lib/mysql/mysql.sock

log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid


lower_case_table_names=1 # 0：区分大小写，1：不区分大小写
default-storage-engine=INNODB
slow_query_log=on
slow_query_log_file=/data/mysql/data/slow_query_log.log
long_query_time=2

# character-set
# # ignore character set information sent by the client
character-set-client-handshake = FALSE
character-set-server = utf8mb4
init_connect='SET NAMES utf8mb4'

max_connections        = 1000
server_id=1

# follow your memory
innodb_buffer_pool_size = 4096M 

# mysql model
sql_mode=STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION


# 8.0 使用新的加密方式，如需兼容，如下配置
# old password plugin
# https://dev.mysql.com/doc/refman/8.0/en/upgrading-from-previous-series.html#upgrade-caching-sha2-password
default_authentication_plugin=mysql_native_password


[client]
default-character-set = utf8mb4
[mysql]
default-character-set = utf8mb4

```

### 启动
如果不想设置 SELinux， 就把 SELinux 关闭，否则会 `(OS errno 13 - Permission denied)`

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

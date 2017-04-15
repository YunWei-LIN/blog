title: activemq (一) 简介
date: 2015-09-11 19:59:44
tags: activemq
categories: mq
---
以下全是linux系统上的操作，在opensuse 13.2 上实际操作。
## 准备
* JDK 1.6 以上
* JAVA_HOME 配置好
* [官网](http://activemq.apache.org/)


## 安装
* 1 下载 activemq gzip 文件
```
$ wget http://ftp.meisei-u.ac.jp/mirror/apache/dist/activemq/5.12.0/apache-activemq-5.12.0-bin.tar.gz
```

* 2 解压缩
```
$ tar zxvf activemq-x.x.x.tar.gz
```

* 3 确认权限
确认activemq 是否有执行权限
```
$ cd [activemq_install_dir]/bin
$ chmod 755 activemq
```




## 启动
进入安装目录的bin路径，启动activemq
```
$ cd [activemq_install_dir]/bin
$ ./activemq start 
```

正常输出
```
INFO: Loading '[activemq_install_dir]/bin/env'
INFO: Using java '/usr/lib64/jvm/java/bin/java'
INFO: Starting - inspect logfiles specified in logging.properties and log4j.properties to get details
INFO: pidfile created : '[activemq_install_dir]/data/activemq.pid' (pid 'xxxx')
```



## 监控
网页监控 [http://localhost:8161/admin](http://localhost:8161/admin)
默认用户密码 admin/admin

可以在 conf/jetty-real.properties 中修改


## 端口
* 网页监控
端口： 8161
配置： [activemq_install_dir]/conf/jetty.xml 

* activeMq 服务
openwire： 61616
amqp    ： 5672
stomp   ： 61613
mqtt    ： 1883
ws      ： 61614
配置： [activemq_install_dir]/conf/activemq.xml 


## 停止
```
$ cd [activemq_install_dir]/bin
$ ./activemq stop 
```

## 配置
[activemq_install_dir]/conf/activemq.xml 


## 集群
[集群](http://activemq.apache.org/clustering.html)




## web demo
官网说 启动 activemq 后， [http://localhost:8161/demo](http://localhost:8161/demo)直接能访问， 其实5.12.0版本是不行的，不行的，不行的。

需要在 conf/jetty 中增加配置

```
<bean class="org.eclipse.jetty.webapp.WebAppContext">
    <property name="contextPath" value="/admin" />
    <property name="resourceBase" value="${activemq.home}/webapps/admin" />
    <property name="logUrlOnStart" value="true" />
</bean>


<!-- 需要增加 -->
<bean class="org.eclipse.jetty.webapp.WebAppContext">
    <property name="contextPath" value="/demo" />
    <property name="resourceBase" value="${activemq.home}/webapps-demo/demo/" />
    <property name="logUrlOnStart" value="true" />
</bean>


...

```
重启activemq， [http://localhost:8161/demo](http://localhost:8161/demo)可以访问了， enjoy it ！




## [activemq集合贴](http://giveme5.top/2015/09/11/mq/mq/#ActiveMQ)

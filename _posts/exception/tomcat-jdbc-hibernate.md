title: tomcat-jdbc hibernate 版本冲突
date: 2015-10-29 16:10:08
tags: [hibernate, tomcat-jdbc, tomcat]  
categories: exception
---
搭建了一套 Spring 核心的 web 服务基础框架。
涉及 Spring 4.2.1.RELEASE ， Spring MVC 4.2.1.RELEASE ，Spring Data JPA 1.9.0.RELEASE， Hibernate 5.0.2.Final， 数据库连接池 tomcat-jdbc 8.27， 容器 tomcat 8.21。

坑人的异常：
```
org.apache.tomcat.jdbc.pool.ConnectionPool abandon
    WARNING: Connection has been abandoned PooledConnection[com.mysql.jdbc.JDBC4Connection@1ad2916]:java.lang.Exception
        at org.apache.tomcat.jdbc.pool.ConnectionPool.getThreadDump(ConnectionPool.java:967)
        at org.apache.tomcat.jdbc.pool.ConnectionPool.borrowConnection(ConnectionPool.java:721)
        at org.apache.tomcat.jdbc.pool.ConnectionPool.borrowConnection(ConnectionPool.java:579)
        at org.apache.tomcat.jdbc.pool.ConnectionPool.getConnection(ConnectionPool.java:174)
        at org.apache.tomcat.jdbc.pool.DataSourceProxy.getConnection(DataSourceProxy.java:111)
        at com.getsom.getConnection(DAO.java:1444)
        at com.getsom.PreparedConnection.(PreparedConnection.java:48)
        at com.getsom.Alarms.run(Alarms.java:492)
```

奇怪的异常， 折腾半天，把 tomcat-jdbc 的配置属性玩了个遍，没辙， 想起以前版本没问题， Hibernate 5.0.2.Final 降级成 Hibernate 4.3.11.Final;
然后就好了！！！！！！
似乎是 对 Hibernate 5 的支持还不够广泛。

ps 推荐一个 [tomcat 8 中文文档](http://wiki.jikexueyuan.com/project/tomcat/), 翻译的还可以。

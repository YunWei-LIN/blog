title: web 服务器
date: 2015-09-16 19:44:58
tags: server
categories: server
---

整理一些常用的服务器知识，配置和使用实例。这里仅作简单介绍。
陆续更新。。。

* tomcat
使用最广泛的 Web 应用服务器和 servlet 容器，不多说了。

* tomEE（发音同“tommy”）
完全兼容tomcat，从名字看就知道和tomcat关系密切。TomEE仅仅是Tomcat的一个扩展版本，任何能在Tomcat上使用的工具，如像Eclipse WTP一样的IDE工具，全部都能用在TomEE上。 TomEE=Tomcat+java EE，TomEE嵌入了EJB、CDI和其他JavaEE特征到Tomcat里，是一个完整符合Web Profile的服务器。

* nginx
是一个高性能的 HTTP 和 反向代理 服务器，也是一个 IMAP/POP3/SMTP 服务器。基本用的是前面的功能 ：)


* wildfly（Jboss as）
JBoss Application Server(JBoss AS) 改名成 wildfly， 版本升级真快。提供完整的Java EE 栈。开源免费。

* jetty
开源的servlet容器
title: Spring JMS (一)
date: 2015-08-26 19:53:49
tags: Spring JMS
categories: spring
---
## JMS
JMS（Java Messaging Service）是Java平台上有关面向消息中间件(MOM)的技术规范.

* 组成
JMS提供者
连接面向消息中间件的，JMS接口的一个实现。提供者可以是Java平台的JMS实现，也可以是非Java平台的面向消息中间件的适配器。
  
JMS客户
生产或消费基于消息的Java的应用程序或对象。
  
JMS生产者（Message Producer）
创建并发送消息的JMS客户。
  
JMS消费者（Message Consumer）
接收消息的JMS客户。
  
JMS消息
包括可以在JMS客户之间传递的数据的对象
  
JMS队列
一个容纳那些被发送的等待阅读的消息的区域。与队列名字所暗示的意思不同，消息的接受顺序并不一定要与消息的发送顺序相同。一旦一个消息被阅读，该消息将被从队列中移走。
  
JMS主题
一种支持发送消息给多个订阅者的机制。
  
  

* 对象模型
连接工厂。连接工厂（ConnectionFactory）是由管理员创建，并绑定到JNDI树中。客户端使用JNDI查找连接工厂，然后利用连接工厂创建一个JMS连接。
  
JMS连接。JMS连接（Connection）表示JMS客户端和服务器端之间的一个活动的连接，是由客户端通过调用连接工厂的方法建立的。
  
JMS会话。JMS会话（Session）表示JMS客户与JMS服务器之间的会话状态。JMS会话建立在JMS连接上，表示客户与服务器之间的一个会话线程。
  
JMS目的。JMS目的（ Destination ），又称为消息队列，是实际的消息源。可以是 queue 或 topic 。
  
JMS生产者和消费者。生产者（Message Producer）和消费者（Message Consumer）对象由Session对象创建，用于发送和接收消息。
  
  

* 消息模型
点对点（Point-to-Point）。在点对点的消息系统中，消息分发给一个单独的使用者。点对点消息往往与队列（javax.jms.Queue）相关联。
发布/订阅（Publish/Subscribe）。发布/订阅消息系统支持一个事件驱动模型，消息生产者和消费者都参与消息的传递。生产者发布事件，而使用者订阅感兴趣的事件，并使用事件。该类型消息一般与特定的主题（javax.jms.Topic）关联。


## Spring JMS
Spring 提供类似JDBC的 JMS集成框架来简单得使用 JMS API。

### JmsTemplate
`JmsTemplate` 是 JMS 核心包的中心类。它简化了JMS的使用，因为它处理了发送或**同步接收消息**时资源的创建和释放。



### Connections
JmsTemplate的需要一个ConnectionFactory的引用。ConnectionFactory是JMS规范的一部分，并作为JMS切入点。

* SingleConnectionFactory
SingleConnectionFactory会在所有 createConnection() 调用时返回相同的 Connection 。

* CachingConnectionFactory
CachingConnectionFactory继承SingleConnectionFactory，并增加了会话，MessageProducer，和MessageConsumers的缓存

### Destination Management


### Message Listener Containers
用来从JMS消息队列接收消息并驱动已经注入的 `MessageListener`，负责所有线程消息的接收和分发到监听器的进程，是MDP和消息提供者之间的中介，并注册接收的消息，参与事务，资源获取和释放，异常转换等等。使程序开发人员可以编写和接收消息相关的（可能比较复杂）业务逻辑。
有以下两种监听器

* SimpleMessageListenerContainer
简单，但不兼容JavaEE的JMS规范


* DefaultMessageListenerContainer
最常用的。XA transaction（JtaTransactionManager）， 可以自定义缓存，可以回复（默认是每5秒，可以自己实现`BackOff`来制定更细的粒度，参考`ExponentialBackOff`）

### Transaction management
Spring提供了JmsTransactionManager来为单独的ConnectionFactory管理事务。
JmsTransactionManager 来管理本地事务和资源。

JtaTransactionManager 来处理分布式事务。


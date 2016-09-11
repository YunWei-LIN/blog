title: activemq （二）  集成Spring-远程调用
date: 2015-09-18 20:34:05
tags: activemq
categories: mq
---

上篇介绍了 [activemq 的安装和使用](http://giveme5.top/2015/09/11/mq/activemq/)，这篇介绍下集成 spring 最简单的运用，远程调用。注意，这种方式是没有事务控制的。

## 加入activemq 的 jar
maven 方式
```
  <dependency>
      <groupId>org.apache.activemq</groupId>
      <artifactId>activemq-all</artifactId>
      <version>5.12.0</version>
  </dependency>
```


## 辅助类

* 接口（服务端和客户端都要有）
```
public interface IGmGameAchvService
{
	/**
	 * 根据ID获取对象
	 * @param companyId
	 * @return
	 */
	GmGameAchvEntity findById(Long id);
}
```

* 实现（服务端）
```
@Service("GmGameAchvServiceImpl")
public class GmGameAchvServiceImpl implements IGmGameAchvService
{

    @Override
    public GmGameAchvEntity findById(Long id)
    {
	GmGameAchvEntity achv = new GmGameAchvEntity();
	achv.setId(1L);
        return achv;
    }
}
```

## 配置 
### 概要
如下内容服务端和客户端都需要配置
1 jmsConnectionFactory
2 jmsQueue

服务端另需配置
3 MessageListenerContainer
4 ServiceExporter

客户端另需配置
5 ProxyFactoryBean

### 具体内容
以下都是java based的配置，xml的配置可以参考 [Spring doc](http://docs.spring.io/spring/docs/4.2.2.BUILD-SNAPSHOT/spring-framework-reference/htmlsingle/#remoting-jms)

* jmsConnectionFactory  jmsQueue

```
@Configuration
public class JmsConfiguration
{

    @Bean
    public ActiveMQConnectionFactory jmsConnectionFactory(){
        ActiveMQConnectionFactory connectionFactory = new ActiveMQConnectionFactory();
        connectionFactory.setBrokerURL("tcp://localhost:61616");
        return connectionFactory;
    }


    @Bean
    public ActiveMQQueue jmsQueue(){
        ActiveMQQueue queue = new ActiveMQQueue("spring-activemq");
        return queue;
    }
}
```


* 服务端
```
@Configuration
public class JmsServerConfiguration
{

    @Autowired
    private IGmGameAchvService gameAchvService;

    @Autowired
    private ActiveMQConnectionFactory jmsConnectionFactory;

    @Autowired
    private ActiveMQQueue jmsQueue;

    @Bean
    public JmsInvokerServiceExporter service()
    {
        JmsInvokerServiceExporter service = new JmsInvokerServiceExporter();
        service.setServiceInterface(IGmGameAchvService.class);
        service.setService(gameAchvService);

        return service;
    }

    @Bean
    public SimpleMessageListenerContainer msgListenerContainer()
    {
        SimpleMessageListenerContainer msgListenerContainer = new SimpleMessageListenerContainer();
        msgListenerContainer.setConnectionFactory(jmsConnectionFactory);
        msgListenerContainer.setDestination(jmsQueue);
        msgListenerContainer.setConcurrentConsumers(3);
        msgListenerContainer.setMessageListener(service());
        return msgListenerContainer;
    }
}

```


* 客户端
配置
```
@Configuration
public class JmsClientConfiguration
{
    @Autowired
    private ActiveMQConnectionFactory jmsConnectionFactory;

    @Autowired
    private ActiveMQQueue jmsQueue;

    @Bean
    public JmsInvokerProxyFactoryBean achvService()
    {
        JmsInvokerProxyFactoryBean service = new JmsInvokerProxyFactoryBean();
        service.setServiceInterface(IGmGameAchvService.class);
        service.setConnectionFactory(jmsConnectionFactory);
        service.setQueue(jmsQueue);
        return service;
    }

}
```

调用
```
  IGmGameAchvService gmservice = (IGmGameAchvService) service;
  GmGameAchvEntity achv = gmservice.findById(1L);
  if (achv == null)
  {
      System.out.println("achv is null");
  }
  else
  {
      System.out.println("achv id=" + achv.getId());
  }
        
```



## 监控
正常启动后， 在 activemq 的监控中可以看到相应内容。
如
[Queues](http://localhost:8161/admin/queues.jsp)
[connections](http://localhost:8161/admin/connections.jsp)



## [activemq集合贴](http://giveme5.top/2015/09/11/mq/mq/#ActiveMQ)


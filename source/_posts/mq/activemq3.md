title: activemq(三) Spring集成 - 消息驱动
date: 2015-09-25 21:22:22
tags: activemq
categories: mq
---
上篇介绍了 [集成Spring-远程调用](), 本次介绍集成 Spring， 常规JMS，消息驱动。主角是 activeMq， JmsTemplate， @JmsListener。
* activeMq 是消息总线，JMS provider； 
* JmsTemplate 是由 Spring提供的消息模板，简化消息生产者和消费者端的代码开发量。就是封装了 连接JMS provider（如ActiveMQ），建立JMS Session（如QueueSession），建立消息生产者，建立消费者，发送消息，接收消息；
* @JmsListener 由 Spring 提供的最简单的消息接收端操作方式。

## 准备
### 必须的jar
maven管理的方式
```
  <dependency>
      <groupId>org.apache.activemq</groupId>
      <artifactId>activemq-all</artifactId>
      <version>5.12.0</version>
  </dependency>
  <dependency>
      <groupId>org.apache.commons</groupId>
      <artifactId>commons-pool2</artifactId>
  </dependency>
```
activemq 的 PooledConnectionFactory 会调用 commons-pool2。

### PropertySourcesPlaceholderConfigurer
配置`PropertySourcesPlaceholderConfigurer`， 下面用得着，比较优雅。
```
@Configuration
public class ApplicationConfiguration
{
    //To resolve ${} in @Value (in *.properties files by @PropertySource included)
    @Bean
    public static PropertySourcesPlaceholderConfigurer propertyConfigInDev() {
      return new PropertySourcesPlaceholderConfigurer();
    }
}
```


## 配置
消息生产者和消费者都需要。
```
import org.apache.activemq.ActiveMQConnectionFactory;
    import org.apache.activemq.command.ActiveMQQueue;
import org.apache.activemq.pool.PooledConnectionFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.PropertySource;
import org.springframework.core.env.Environment;
import org.springframework.jms.annotation.EnableJms;
import org.springframework.jms.config.DefaultJmsListenerContainerFactory;
import org.springframework.jms.connection.CachingConnectionFactory;
import org.springframework.jms.core.JmsTemplate;

@Configuration
@EnableJms //开启jsm 注解
@PropertySource({ "classpath:conf/jms.properties"}) // 加载配置资源文件
public class JmsConfiguration
{

    @Autowired
    private Environment env;

    //ConnectionFactory
    @Bean
    public ActiveMQConnectionFactory jmsConnectionFactory()
    {
        ActiveMQConnectionFactory connectionFactory = new ActiveMQConnectionFactory();
        connectionFactory.setBrokerURL("tcp://localhost:61616");
        return connectionFactory;
    }

    //activeMq PooledConnectionFactory
    @Bean
    public PooledConnectionFactory activeMqPooledConnectionFactory()
    {
        PooledConnectionFactory jmsFactory = new PooledConnectionFactory();
        jmsFactory.setConnectionFactory(jmsConnectionFactory());
        return jmsFactory;
    }

    //spring CachingConnectionFactory
    @Bean
    public CachingConnectionFactory springCachingConnectionFactory()
    {
        CachingConnectionFactory jmsFactory = new CachingConnectionFactory();
        jmsFactory.setTargetConnectionFactory(jmsConnectionFactory());
        return jmsFactory;
    }

    // Queue
    @Bean
    public ActiveMQQueue jmsQueue()
    {
        ActiveMQQueue queue = new ActiveMQQueue(env.getProperty("testQueue"));
        return queue;
    }

    // ListenerContainerFactory， @EnableJms 需要这个默认 bean
    @Bean
    public DefaultJmsListenerContainerFactory jmsListenerContainerFactory() {
        DefaultJmsListenerContainerFactory factory =
                new DefaultJmsListenerContainerFactory();
        factory.setConnectionFactory(activeMqPooledConnectionFactory());
        factory.setConcurrency("3-10");
        return factory;
    }

    // Spring jmsTemplate
    @Bean
    public JmsTemplate myJmsTemplate()
    {
        JmsTemplate jmsTp = new JmsTemplate();
        jmsTp.setDefaultDestination(jmsQueue());
        jmsTp.setConnectionFactory(activeMqPooledConnectionFactory());
        return jmsTp;
    }
}

```

补充说明：
* @EnableJms：开启 JMS 注解， 这样 @JmsListener 等注解才会生效；默认需要jmsListenerContainerFactory 这个 Bean。
* activeMq PooledConnectionFactory 和 springCachingConnectionFactory 同等作用，可以替换。
* testQueue 具体value 在 jms.properties 里配置，类似常量。


## 消息发送和接收
首先看代码
```
@Service("GmGameAchvServiceImpl")
@PropertySource({ "classpath:conf/jms.properties"})
public class GmGameAchvServiceImpl
{
    @Resource(name = "myJmsTemplate")
    private JmsTemplate jmsTemplate;


    // 消息接收
    @JmsListener(destination = "${testQueue}")
    public void JmsReceiveTest(String data)
    {
        System.out.println("jms ----------------------------- " + data);
    }


    //消息发送
    public void jmsSendTest()
    {
        int numberOfMessages = 10;
        StringBuilder payload = null;
        for (int i = 0; i < numberOfMessages; ++i)
        {
            payload = new StringBuilder();
            payload.append("Message [").append(i).append("] sent at: ").append(new Date());
            final int finalI = i;
            final StringBuilder finalPayload = payload;
            jmsTemplate.send((Session session) -> {
                TextMessage message = session.createTextMessage(finalPayload.toString());
                message.setIntProperty("messageCount", finalI);
                return message;
            });


        }
    }
```

* 消息发送
方法 `jmsSendTest`， 利用 jmsTemplate 很方便
* 消息接收
`JmsReceiveTest`， 使用了 `@JmsListener`，这是最简单的方式，比 jmsTemplate 更简单。
`@JmsListener` 需要制定 `destination`， 这里 `${testQueue}` 的值由 `conf/jms.properties` 里具体制定， 由上文提到的 `PropertySourcesPlaceholderConfigurer` 负责转化。 如此在一个配置文件里配置，多个地方使用，如需改动，仅需修改配置文件，不用修改源代码。


## 消息数据转换 Message Converters
Spring 会自动进行消息转换， 比如上面 `jmsSendTest` 发送的是 `TextMessage`， `JmsReceiveTest` 用 String 类型参数接收就好。POJO 对象一样可以如此操作。
```
    @Override
    @JmsListener(destination = "${testQueue}")
    public void JmsReceiveTest(GmGameAchvEntity data)
    {
        System.out.println("jms ----------------------------- " + data.getId());
    }

    @Override
    public void jmsSendTest()
    {
        int numberOfMessages = 10;
        StringBuilder payload = null;
        for (int i = 0; i < numberOfMessages; ++i)
        {
            final int finalI = i;
            GmGameAchvEntity achv = new GmGameAchvEntity();
            achv.setId((long) i);
            jmsTemplate.send((Session session) -> {
                ObjectMessage message = session.createObjectMessage(achv);
                message.setIntProperty("messageCount", finalI);
                return message;
            });
        }
    }
```
如此这般就好了，是不是 很简单方便。


## 响应管理
```
@JmsListener(destination = "myQueue")
@SendTo("queueOut")
public OrderStatus processOrder(Order order) {
    // order processing
    return status;
}
```
`@SendTo` 就可以返回值 到 `queueOut` 这个destination


～～～～～～～ 嗯嗯， 很不错的样子 ：）




## [activemq集合贴](http://giveme5.top/2015/09/11/mq/mq/#ActiveMQ)
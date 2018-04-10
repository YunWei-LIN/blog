title: Spring JMS （二）
date: 2015-09-6 18:53:49
tags: [spring,  JMS]
categories: spring
---
## 发送 JMS 消息
### 简单的使用默认destination

```
import javax.jms.ConnectionFactory;
import javax.jms.JMSException;
import javax.jms.Message;
import javax.jms.Queue;
import javax.jms.Session;

import org.springframework.jms.core.MessageCreator;
import org.springframework.jms.core.JmsTemplate;

public class JmsQueueSender {

    private JmsTemplate jmsTemplate;
    private Queue queue;

    public void setConnectionFactory(ConnectionFactory cf) {
        this.jmsTemplate = new JmsTemplate(cf);
    }

    public void setQueue(Queue queue) {
        this.queue = queue;
    }

    public void simpleSend() {
        this.jmsTemplate.send(this.queue, new MessageCreator() {
            public Message createMessage(Session session) throws JMSException {
                return session.createTextMessage("hello queue world");
            }
        });
    }
}
```

###  Message Converters
 Message Converters 提供Java 对象 message‘s 数据间的转换。Spring的默认实现 SimpleMessageConverter 可以支持String 和 TextMessage, byte[] 和 BytesMesssage,  java.util.Map 和 MapMessage 之间的转换。
下面是个Map的发送：
```
public void sendWithConversion() {
    Map map = new HashMap();
    map.put("Name", "Mark");
    map.put("Age", new Integer(47));
    jmsTemplate.convertAndSend("testQueue", map, new MessagePostProcessor() {
        public Message postProcessMessage(Message message) throws JMSException {
            message.setIntProperty("AccountID", 1234);
            message.setJMSCorrelationID("123-00001");
            return message;
        }
    });
}

//This results in a message of the form:
//MapMessage={
//	Header={
//		... standard headers ...
//		CorrelationID={123-00001}
//	}
//	Properties={
//		AccountID={Integer:1234}
//	}
//	Fields={
//		Name={String:Mark}
//		Age={Integer:47}
//	}
//}
```
SessionCallback 回调和 ProducerCallback 回调


## 接收 JMS 消息
### 同步接收 Synchronous
同步接收 JMS 消息会堵塞， 可设置 `receiveTimeout` 。


### 异步接收 Asynchronous - Message-Driven POJOs
类似于 EJB 里的  Message-Driven Bean (MDB) ，Spring 定义了 Message-Driven POJO (MDP) 来作为 JMS 的接收者。
一个 Message-Driven POJO (MDP) 必须实现 `javax.jms.MessageListener` （或者 MessageListenerAdapter， SessionAwareMessageListener），而且必须是线程安全，它会被多线程调用。
 MDP的一个例子：
```
import javax.jms.JMSException;
import javax.jms.Message;
import javax.jms.MessageListener;
import javax.jms.TextMessage;

public class ExampleListener implements MessageListener {

    public void onMessage(Message message) {
        if (message instanceof TextMessage) {
            try {
                System.out.println(((TextMessage) message).getText());
            }
            catch (JMSException ex) {
                throw new RuntimeException(ex);
            }
        }
        else {
            throw new IllegalArgumentException("Message must be of type TextMessage");
        }
    }

}
```
对应的配置
```
<!-- this is the Message Driven POJO (MDP) -->
<bean id="messageListener" class="jmsexample.ExampleListener" />

<!-- and this is the message listener container -->
<bean id="jmsContainer" class="org.springframework.jms.listener.DefaultMessageListenerContainer">
    <property name="connectionFactory" ref="connectionFactory"/>
    <property name="destination" ref="destination"/>
    ---<property name="messageListener" ref="messageListener" />---
</bean>
```

### 事务 transaction
本地事务只需要简单配置 `sessionTransacted` 就可以激活。发送响应是该本地事务的一部分，但其他所有资源（如数据库操作）的操作都是独立的。
```
<bean id="jmsContainer" class="org.springframework.jms.listener.DefaultMessageListenerContainer">
    <property name="connectionFactory" ref="connectionFactory"/>
    <property name="destination" ref="destination"/>
    <property name="messageListener" ref="messageListener"/>
    <property name="sessionTransacted" value="true"/>
</bean>
```

分布式事务
```
<bean id="transactionManager" class="org.springframework.transaction.jta.JtaTransactionManager"/>

<bean id="jmsContainer" class="org.springframework.jms.listener.DefaultMessageListenerContainer">
    <property name="connectionFactory" ref="connectionFactory"/>
    <property name="destination" ref="destination"/>
    <property name="messageListener" ref="messageListener"/>
    <property name="transactionManager" ref="transactionManager"/>
</bean>

```



## 注解驱动的监听端点
@JmsListener


### 开启监听端点 注解

在`@Configuration`类中加入`@EnableJms`来使`@JmsListener`生效。
```
@Configuration
@EnableJms
public class AppConfig {

    @Bean
    public DefaultJmsListenerContainerFactory jmsListenerContainerFactory() {   //default name
        DefaultJmsListenerContainerFactory factory =
                new DefaultJmsListenerContainerFactory();
        factory.setConnectionFactory(connectionFactory());
        factory.setDestinationResolver(destinationResolver());
        factory.setConcurrency("3-10"); //pool size 3～10 threads
        return factory;
    }
}
```

### 注册监听端点
```
@Configuration
@EnableJms
public class AppConfig implements JmsListenerConfigurer {

    @Override
    public void configureJmsListeners(JmsListenerEndpointRegistrar registrar) {
        SimpleJmsListenerEndpoint endpoint = new SimpleJmsListenerEndpoint();
        endpoint.setId("myJmsEndpoint");
        endpoint.setDestination("anotherQueue");
        endpoint.setMessageListener(message -> {
            // processing
        });
        registrar.registerEndpoint(endpoint);
    }
}
```

### 端点方法 签名
```
@Component
public class MyService {

    @JmsListener(destination = "myDestination")
    public void processOrder(Order order, @Header("order_type") String orderType) {
        ...
    }
}
```

### 响应管理
`@SendTo`
```
@JmsListener(destination = "myDestination")
@SendTo("status")
public OrderStatus processOrder(Order order) {
    // order processing
    return status;
}


//or

@JmsListener(destination = "myDestination")
@SendTo("status")
public Message<OrderStatus> processOrder(Order order) {
    // order processing
    return MessageBuilder
            .withPayload(status)
            .setHeader("code", 1234)
            .build();
}
```


运行时响应destination
```
@JmsListener(destination = "myDestination")
public JmsResponse<Message<OrderStatus>> processOrder(Order order) {
    // order processing
    Message<OrderStatus> response = MessageBuilder
            .withPayload(status)
            .setHeader("code", 1234)
            .build();
    return JmsResponse.forQueue(response, "status");
}
```




## [JMS 命名空间](http://docs.spring.io/spring/docs/4.2.2.BUILD-SNAPSHOT/spring-framework-reference/htmlsingle/#jms-namespace)


## [Spring 集成 ActiveMq](http://giveme5.top/2015/09/25/mq/activemq3/)

title: spring Transaction 事务
date: 2017-11-03 19:44:58
tags: [spring, transaction]
categories: spring
---

## 主要内容
spring Transaction 事务；Transaction 事务绑定事件

*更新历史*
无

环境：spring 4.2 以上

spring 的 事务分为 `编程式事务` 和 `声明式事务` 。
对此， spring 官方的建议是： 除非你只有很少数目的事务操作， 你可以选择 `编程式事务` ；否则都建议用 `声明式事务`

<!-- more -->

---

## 编程式事务
spring 提供2种方式：
+ `TransactionTemplate`
+ 直接使用 `PlatformTransactionManager`

### TransactionTemplate

### 直接 PlatformTransactionManager
实例化 `PlatformTransactionManager` 的实现类(比如 `DataSourceTransactionManager`) 得到 `txManager` , 需要一个 datasource；
然后 `DefaultTransactionDefinition` 定义事务， `TransactionStatus` 操作具体SQL。

```java
DefaultTransactionDefinition def = new DefaultTransactionDefinition();

// explicitly setting the transaction name is something that can only be done programmatically
def.setName("SomeTxName");
def.setPropagationBehavior(TransactionDefinition.PROPAGATION_REQUIRED);

TransactionStatus status = txManager.getTransaction(def);
try {
        // execute your business logic here
}
catch (MyException ex) {
        txManager.rollback(status);
        throw ex;
}
txManager.commit(status);
```

### TransactionTemplate

```java
public class SimpleService implements Service {

    private final TransactionTemplate transactionTemplate;

    public SimpleService(PlatformTransactionManager transactionManager) {
        Assert.notNull(transactionManager, "The 'transactionManager' argument must not be null.");
        this.transactionTemplate = new TransactionTemplate(transactionManager);

        // the transaction settings can be set here explicitly if so desired
        this.transactionTemplate.setIsolationLevel(TransactionDefinition.ISOLATION_READ_UNCOMMITTED);
        this.transactionTemplate.setTimeout(30); // 30 seconds
        // and so forth...
    }
    
    //有返回值
    public Object someServiceMethod() {
        return transactionTemplate.execute(new TransactionCallback() {
                // the code in this method executes in a transactional context
                public Object doInTransaction(TransactionStatus status) {
                        updateOperation1();
                        return resultOfUpdateOperation2();
                }
        });
    }
    
    //无返回值
    public Object someServiceMethod() {
        transactionTemplate.execute(new TransactionCallbackWithoutResult() {
                protected void doInTransactionWithoutResult(TransactionStatus status) {
                        updateOperation1();
                        updateOperation2();
                }
        });
    }
}


```


## 声明式事务
注解方式的 声明式事务 主要涉及 `@Transactional` 和 `@EnableTransactionManagement` (需要配合 `@Configuration`)。
声明式事务 主要通过 事务代理 来实现，如下图：

![](/images/spring_tx.png)
> Transaction Advisor 可由 Spring 提供默认实现；Custom Advisor(s) 可借 Spring 事务绑定事件`@TransactionalEventListener` 触发自定义。

### Transactional
+ 属性

|属性|类型|描述|
|---|---|---|
|value|String|Optional , 指定transaction manager|
|propagation|enum: Propagation| Optional， 指定 传播propagation|
|isolation|enum: Isolation|Optional 可选隔离级别|
|readOnly|boolean|读/写与只读事务|
|timeout|int (in seconds granularity)|事务超时|


#### propagation 事务传播
+ PROPAGATION_REQUIRED 默认设置
Support a current transaction, create a new one if none exists.
![](/images/spring_tx_prop_required.png)

+ PROPAGATION_REQUIRES_NEW
Create a new transaction, and suspend the current transaction if one exists.
![](/images/spring_tx_prop_requires_new.png)

### 事务绑定事件
```java
@Component
public class MyComponent {

    @TransactionalEventListener
    public void handleOrderCreatedEvent(CreationEvent<Order> creationEvent) {
                ...
    }
}
```
`@TransactionalEventListener` 有个属性 `phase` (BEFORE_COMMIT, AFTER_COMMIT (default), AFTER_ROLLBACK and AFTER_COMPLETION)

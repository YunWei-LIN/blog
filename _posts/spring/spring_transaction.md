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

<style>
table th:first-of-type {
    width: 20%;
}
table th:nth-of-type(2) {
    width: 20%;
}
</style>

|属性|类型|描述|
|---|---|---|
|value|String|Optional , 指定transaction manager|
|propagation|enum: Propagation| Optional， 指定 传播propagation|
|isolation|enum: Isolation|Optional 可选隔离级别|
|readOnly|boolean|读/写与只读事务|
|timeout|int (in seconds granularity)|事务超时|
|rollbackFor|Class|触发事务回滚的类, Throwable 或 Throwable的子类，默认是对unchecked异常（ERORR和RuntimeException）起作用|
|rollbackFor<br>ClassName|String|设置需要进行回滚的异常类名称数组，当方法中抛出指定异常名称数组中的异常时，则进行事务回滚|
|noRollbackFor|Class|设置不需要进行回滚的异常类数组，当方法中抛出指定异常数组中的异常时，不进行事务回滚。Throwable 或 Throwable的子类|
|noRollbackFor<br>ClassName|String|设置不需要进行回滚的异常类名称数组，当方法中抛出指定异常名称数组中的异常时，不进行事务回滚。|


#### propagation 事务传播
+ PROPAGATION_REQUIRED 默认设置
如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务。
Support a current transaction, create a new one if none exists.
![](/images/spring_tx_prop_required.png)

+ PROPAGATION_REQUIRES_NEW
创建一个新的事务，如果当前存在事务，则把当前事务挂起。
Create a new transaction, and suspend the current transaction if one exists.
![](/images/spring_tx_prop_requires_new.png)

+ PROPAGATION_SUPPORTS
如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。

+ PROPAGATION_NOT_SUPPORTED
以非事务方式运行，如果当前存在事务，则把当前事务挂起。

+ PROPAGATION_MANDATORY
如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常。

+ PROPAGATION_NEVER
以非事务方式运行，如果当前存在事务，则抛出异常

+ PROPAGATION_NESTED
如果当前存在事务，则创建一个事务作为当前事务的嵌套事务（一个事务中可以包括多个保存点，每一个保存点嵌套子事务）来运行； 如果当前没有事务，则该取值等价于TransactionDefinition.PROPAGATION_REQUIRED。

#### 隔离级别
+ ISOLATION_DEFAULT
这是默认值，表示使用底层数据库的默认隔离级别。对大部分数据库而言，通常这值就是READ_COMMITTED, MySQL是REPEATABLE_READ。

+ ISOLATION_READ_UNCOMMITTED
该隔离级别表示一个事务可以读取另一个事务修改但还没有提交的数据。该级别不能防止脏读和不可重复读，因此很少使用该隔离级别。

+ ISOLATION_READ_COMMITTED
该隔离级别表示一个事务只能读取另一个事务已经提交的数据。该级别可以防止脏读，这也是大多数情况下的推荐值。

+ ISOLATION_REPEATABLE_READ
该隔离级别表示一个事务在整个过程中可以多次重复执 行某个查询，并且每次返回的记录都相同。即使在多次查询之间有新增的数据满足该查询，这些新增的记录也会被忽略。该级别可以防止脏读和不可重复读。

+ ISOLATION_SERIALIZABLE
所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，该级别可以防止脏读、不可重复读以及幻读。但是这将严重影响程序的性能。通常情况下也不会用到该级别。


### Transactional注意点
特殊，单独提高等级来写

#### Transactional特性
+ service实现类标签
在service实现类 类头(一般不建议在接口上)上添加@Transactional，可以将整个类纳入spring事务管理，在每个业务方法执行时都会开启一个事务，不过这些事务采用相同的管理方式。

+ 可见度
@Transactional 注解只能应用到 public 可见度的方法上。 如果应用在protected、private或者 package可见度的方法上，也不会报错，不过事务设置不会起作用。

+ 回滚
默认情况下，spring会对unchecked异常进行事务回滚；如果是checked异常则不回滚。 
通俗一点：你写代码出现的空指针等异常，会被回滚，文件读写，网络出问题，spring就没法回滚了。
> java里面将派生于Error或者RuntimeException（比如空指针，1/0）的异常称为unchecked异常，其他继承自java.lang.Exception得异常统称为Checked Exception，如IOException、TimeoutException 

+ 只读事务
只读标志只在事务启动时应用，否则即使配置也会被忽略。 
启动事务会增加线程开销，数据库因共享读取而锁定(具体跟数据库类型和事务隔离级别有关)。通常情况下，仅是读取数据时，不必设置只读事务而增加额外的系统开销。

#### 解决Transactional不回滚
+ 检查方法是否是`public`的。
+ 异常类型是否是`unchecked`异常。
如果想check异常也想回滚怎么办，注解上面写明异常类型即可。
```java
@Transactional(rollbackFor=Exception.class)
```

+ `spring`是否开启对注解的解析
`@EnableTransactionManagement` 
还有例如SpringDataJPA 事务容器声明： 
transactionManager(JpaTransactionManager) -> entityManagerFactory(EntityManagerFactory) -> dataSource

+ `spring`是否扫描到包

+ 数据库引擎是否支持事务

### 事务绑定事件
`@TransactionalEventListener`

```java
@Service
public class TransactionEventTestService {

    @Resource
    private TestMapper mapper;

    @Resource
    private ApplicationEventPublisher publisher;

    @Transactional
    public void addTestModel() {
        TestModel model = new TestModel();
        model.setName("haogrgr");
        mapper.insert(model);

        //如果model没有继承ApplicationEvent, 则内部会包装为PayloadApplicationEvent
        //对于@TransactionalEventListener, 会在事务提交后才执行Listener处理逻辑.
        
        //发布事件, 事务提交后, 记录日志, 或发送消息等操作
        publisher.publishEvent(model);
    }
    //当事务提交后, 才会真正的执行@TransactionalEventListener配置的Listener, 如果Listener抛异常, 方法返回失败, 但事务不会回滚.

}

@Component
public class TransactionEventListener {

    @TransactionalEventListener
    public void handle(PayloadApplicationEvent<TestModel> event) {
        System.out.println(event.getPayload().getName());
        //这里可以记录日志, 发送消息等操作.
        //这里抛出异常, 会导致addTestModel方法异常, 但不会回滚事务.
        //注意, ApplicationEventPublisher不能使用线程池, 否则不会执行到这里
        //因为, 包装类是通过ThreadLocal来判断当前是否有活动的事务信息.
        //TransactionalEventListener.fallbackExecution就是为了决定当当前线程没有事务上下文时,
        //是否还调用 handle 方法, 默认不调用.
    }
}
```
#### 属性 
+ `phase` 
BEFORE_COMMIT, AFTER_COMMIT (default), AFTER_ROLLBACK and AFTER_COMPLETION

+ condition

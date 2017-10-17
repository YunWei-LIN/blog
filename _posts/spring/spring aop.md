title: spring aop 面向切面编程
date: 2017-10-13 19:44:58
tags: [spring, aop]
categories: spring
---

## 主要内容
spring aop 面向切面编程; 主要探索Spring框架对面向切面编程的支持，包括如何定义需要被切面(aspect)覆盖的类，如何使用注解创建切面。

* AOP基本知识
* spring AOP

*更新历史*
无

<!-- more -->

---------------------------------------------------------------------


## AOP基本知识
面向切面编程(AOP)模块化的单元则是切面。切面能对交叉关注点进行模块化，简单来说，交叉关注点值得是那些影响一个应用中多个模块的通用功能。

![](/images/aop1.png)

把交叉关注点模块化到某个特定的类，这个类就称为切面(aspects)，这有两个优点：
关注点分离，而不是与业务逻辑代码混合在一起；
业务模块更加清晰，因为它们只需要关注业务逻辑部分；


## AOP 术语


+ 切面(Aspect)
+ 连接点(Joinpoint)
+ 切入点(Pointcut)
+ 引入(Introduction)
+ 目标对象(Target Object)
+ AOP代理(AOP Proxy)
+ 织入(Weaving)
+ 通知(Advice)
    通知类型：
    + 前置通知(Before advice)：在某连接点之前执行的通知
    + 后置通知(After returning advice)：在某连接点正常完成后执行的通知
    + 异常通知(After throwing advice)：在方法抛出异常退出时执行的通知
    + 最终通知(After (finally) advice)当某连接点退出的时候执行的通知(不论是正常返回还是异常退出)
    + 环绕通知(Around Advice)：包围一个连接点的通知，如方法调用。这是最强大的一种通知类型。环绕通知可以在方法调用前后完成自定义的行为。它也会选择是否继续执行连接点或直接返回它自己的返回值或抛出异常来结束执行。

### 切面(Aspect)
一个关注点的模块化，这个关注点可能会横切多个对象。通常由 通知(advice)、切点(pointcuts)和织入点(join points)组成；
事务管理是J2EE应用中一个关于横切关注点的很好的例子。在Spring AOP中，切面可以使用基于模式)或者基于@Aspect注解的方式来实现。
![](/images/aop2.png)
>>切面的功能(advice)通过一个或者多个织入点织入到应用的执行流程

### 连接点(Joinpoint)
在程序执行过程中某个特定的点，可能是正在调用的方法、正在抛出的异常或者是正在被修改的属性, 比如某方法调用的时候或者处理异常的时候。
在Spring AOP中，一个连接点总是表示一个方法的执行。
通常是业务代码中的概念， 不在切面(Aspect)代码中体现。

### 切入点(Pointcut)
匹配连接点的断言， 逻辑上是连接点的子集，就是当前切面(Aspect)所关心/处理的连接点。
在切面(Aspect)代码中通过 `@Pointcut` 定义。

### 通知(Advice)
在切面的某个特定的切入点上执行的动作， 就是切面的真正目的——它真正要做的工作。其中包括了“around”、“before”和“after”等不同类型的通知(通知的类型将在后面部分进行讨论)。许多AOP框架(包括Spring)都是以拦截器做通知模型，并维护一个以连接点为中心的拦截器链。
通知和一个切入点表达式关联，并在满足这个切入点的连接点上运行(例如，当执行某个特定名称的方法时)。切入点表达式如何和连接点匹配是AOP的核心：Spring缺省使用AspectJ切入点语法。

### 引入(Introduction)
用来给一个类型声明额外的方法或属性(也被称为连接类型声明(inter-type declaration))。Spring允许引入新的接口(以及一个对应的实现)到任何被代理的对象。例如，你可以使用引入来使一个bean实现IsModified接口，以便简化缓存机制。

### 目标对象(Target Object)
被一个或者多个切面所通知的对象。也被称做被通知(advised)对象。 既然Spring AOP是通过运行时代理实现的，这个对象永远是一个被代理(proxied)对象。

### AOP代理(AOP Proxy)
AOP框架创建的对象，用来实现切面契约(例如通知方法执行等等)。在Spring中，AOP代理可以是JDK动态代理或者CGLIB代理。

### 织入(Weaving)
把切面连接到其它的应用程序类型或者对象上，并创建一个被通知的对象。这些可以在编译时(例如使用AspectJ编译器)，类加载时和运行时完成。Spring和其他纯Java AOP框架一样，在运行时完成织入。


## Spring 的AOP支持
Spring对AOP的支持来自以下四种形式：

+ 基于代理的Spring AOP
+ Pure-POJO aspects
+ 基于@AspectJ注解的aspects
+ 注入AspectJ aspects(所有版本的Spring都支持)

前三种属于Spring自己的AOP实现：Spring AOP基于动态代理机制构建，也正是因为这个原因，Spring AOP仅仅支持函数调用级别的拦截。


### 启用@AspectJ支持
java config： `@EnableAspectJAutoProxy`
或
xml config： `<aop:aspectj-autoproxy />`

### 完整(经典)切面
一个 完整(经典)切面 通常 由 切面， 切入点， 通知 3部分组成。


#### 声明一个切面 `@Aspect`
```java
@Aspect
public class Aspect {

}
```

#### 声明一个切入点(pointcut) `@Pointcut`
```java
@Pointcut("execution(* transfer(..))")// the pointcut expression
private void anyOldTransfer() {}// the pointcut signature
```

切入点指示符(PCD)

|切入点指示符|说明|
|---| :------------------------------------------ |
|execution|匹配方法执行的连接点，这是你将会用到的Spring的最主要的切入点指示符。|
|within|限定匹配特定类型的连接点(在使用Spring AOP的时候，在匹配的类型中定义的方法的执行)。|
|this|限定匹配特定的连接点(使用Spring AOP的时候方法的执行)，其中bean reference(Spring AOP 代理)是指定类型的实例。|
|target|限定匹配特定的连接点(使用Spring AOP的时候方法的执行)，其中目标对象(被代理的应用对象)是指定类型的实例。|
|args|限定匹配特定的连接点(使用Spring AOP的时候方法的执行)，其中参数是指定类型的实例。|
|@target|限定匹配特定的连接点(使用Spring AOP的时候方法的执行)，其中正执行对象的类持有指定类型的注解。|
|@args|限定匹配特定的连接点(使用Spring AOP的时候方法的执行)，其中实际传入参数的运行时类型持有指定类型的注解。|
|@within|限定匹配特定的连接点，其中连接点所在类型已指定注解(在使用Spring AOP的时候，所执行的方法所在类型已指定注解)。|
|@annotation|限定匹配特定的连接点(使用Spring AOP的时候方法的执行)，其中连接点的主题持有指定的注解。|

更多内容参考附录

常用的切入点(pointcut)例子 
```java
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Pointcut;

@Aspect
public class SystemArchitecture {

  /**
   * A join point is in the web layer if the method is defined
   * in a type in the com.xyz.someapp.web package or any sub-package
   * under that.
   */
  @Pointcut("within(com.xyz.someapp.web..*)")
  public void inWebLayer() {}

  /**
   * A join point is in the service layer if the method is defined
   * in a type in the com.xyz.someapp.service package or any sub-package
   * under that.
   */
  @Pointcut("within(com.xyz.someapp.service..*)")
  public void inServiceLayer() {}

  /**
   * A join point is in the data access layer if the method is defined
   * in a type in the com.xyz.someapp.dao package or any sub-package
   * under that.
   */
  @Pointcut("within(com.xyz.someapp.dao..*)")
  public void inDataAccessLayer() {}

  /**
   * A business service is the execution of any method defined on a service
   * interface. This definition assumes that interfaces are placed in the
   * "service" package, and that implementation types are in sub-packages.
   * 
   * If you group service interfaces by functional area (for example, 
   * in packages com.xyz.someapp.abc.service and com.xyz.def.service) then
   * the pointcut expression "execution(* com.xyz.someapp..service.*.*(..))"
   * could be used instead.
   *
   * Alternatively, you can write the expression using the 'bean'
   * PCD, like so "bean(*Service)". (This assumes that you have
   * named your Spring service beans in a consistent fashion.)
   */
  @Pointcut("execution(* com.xyz.someapp.service.*.*(..))")
  public void businessService() {}
  
  /**
   * A data access operation is the execution of any method defined on a 
   * dao interface. This definition assumes that interfaces are placed in the
   * "dao" package, and that implementation types are in sub-packages.
   */
  @Pointcut("execution(* com.xyz.someapp.dao.*.*(..))")
  public void dataAccessOperation() {}

}
```

#### 声明通知
+ 前置通知 `@Before` 
声明前置通知

+ 后置通知 `@AfterReturning` 
在一个匹配的方法返回的时候执行

    通知体内得到返回的值
    ```java
    import org.aspectj.lang.annotation.Aspect;
    import org.aspectj.lang.annotation.AfterReturning;

    @Aspect
    public class AfterReturningExample {

    @AfterReturning(
        pointcut="com.xyz.myapp.SystemArchitecture.dataAccessOperation()",
        returning="retVal")
    public void doAccessCheck(Object retVal) {
        // ...
    }
    
    }
    ```

+ 异常通知 `@AfterThrowing` 
只在某种特殊的异常被抛出的时候匹配
还可以将抛出的异常绑定到通知的一个参数上
```java
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.AfterThrowing;

@Aspect
public class AfterThrowingExample {

  @AfterThrowing(
    pointcut="com.xyz.myapp.SystemArchitecture.dataAccessOperation()",
    throwing="ex")
  public void doRecoveryActions(DataAccessException ex) {
    // ...
  }

}
```

+ 最终通知  `@After` 不论一个方法是如何结束的，最终通知都会运行。
最终通知必须准备处理正常返回和异常返回两种情况。通常用它来释放资源。

+ 环绕通知 `@Around`
环绕通知在一个方法执行之前和之后执行。
通知的第一个参数必须是 ProceedingJoinPoint类型。在通知体内，调用 ProceedingJoinPoint的proceed()方法会导致 后台的连接点方法执行。proceed 方法也可能会被调用并且传入一个 Object[]对象-该数组中的值将被作为方法执行时的参数。
```java
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.ProceedingJoinPoint;

@Aspect
public class AroundExample {

  @Around("com.xyz.myapp.SystemArchitecture.businessService()")
  public Object doBasicProfiling(ProceedingJoinPoint pjp) throws Throwable {
    // start stopwatch
    Object retVal = pjp.proceed();
    // stop stopwatch
    return retVal;
  }

}
```

#### 通知参数(Advice parameters)
可以在通知签名中声明所需的参数.

##### 传递参数给通知
定义一个切入点,然后直接从通知中访问那个命名切入点。
如下的参数 `account`

```java
@Pointcut("com.xyz.myapp.SystemArchitecture.dataAccessOperation() &&" + 
          "args(account,..)")
private void accountDataAccessOperation(Account account) {}

@Before("accountDataAccessOperation(account)")
public void validateAccount(Account account) {
  // ...
}
```

##### 确定参数名
如果第一个参数是JoinPoint， ProceedingJoinPoint， 或者JoinPoint.StaticPart类型， 你可以在“argNames”属性的值中省去参数的名字。
其他使用 额外的"argNames"属性指定的参数名。
如下 参数 `bean`, `jp` 都可以使用。

```java
@Before(
   value="com.xyz.lib.Pointcuts.anyPublicMethod() && target(bean) && @annotation(auditable)",
   argNames="bean,auditable")
public void audit(JoinPoint jp, Object bean, Auditable auditable) {
  AuditCode code = auditable.value();
  // ... use code, bean, and jp
}
```

##### 处理参数
带参数的的proceed()调用

```java
@Around("execution(List<Account> find*(..)) &&" +
        "args(accountHolderNamePattern)")		
public Object preProcessQueryPattern(ProceedingJoinPoint pjp, String accountHolderNamePattern)
throws Throwable {
  String newPattern = preProcess(accountHolderNamePattern);
  return pjp.proceed(new Object[] {newPattern});
}
```

#### 完整切面例子
上面章节分别说明了切面，切入点，通知， 下面结合整体例子说明， 注意 1,2,3 点。

```java
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.aspectj.lang.annotation.Pointcut;
import java.util.HashMap;import java.util.Map;

@Aspect //1. 切面
public class TrackCounter {
    private Map<Integer, Integer> trackCounts = new HashMap<Integer, Integer>();

    @Pointcut( //2. 切入点
            "execution(* com.spring.sample.soundsystem.CompactDisc.playTrack( .. )) " +
            "&& args(trackNumber)")  
    public void trackPlayed(int trackNumber) {} //2.1 注意参数声明和上面切入点签名一致

    @Before("trackPlayed(trackNumber)")  //3. 通知
    public void countTrack(int trackNumber) {  //3.1 注意参数声明和上面通知签名一致
        int currentCount = getPlayCount(trackNumber);
        trackCounts.put(trackNumber, currentCount + 1);
    }

    public int getPlayCount(int trackNumber) {
        return trackCounts.containsKey(trackNumber) ？ trackCounts.get(trackNumber) : 0;
    }
}

```

## 引入(Introduction)
在不修改原有类的基础上为该类添加新方法，即通过切面为Spring的beans增加新方法。
使用 `@DeclareParents` 注解

假定现有一个来自Spring的接口 `SpringIntroduction`, 现在你需要给它增加 `Encode`功能， 
```java
public interface Encode {
    void AopEncode();
}
```
你暂时还不能修改Spring 的代码， 那你就可以使用Spring AOP。

### introduced的默认实现
代码如下：
```java
public class DefaultEncode implements Encode {
    public void aopEncode() {
        System.out.println("Introduction the Encode!");
    }
}

```

### introduced 切面
```java
@Aspect
public class EncodeIntroducer {
    @DeclareParents(value = "io.spring.xxx.SpringIntroduction+",
                    defaultImpl = DefaultEncode.class)
    public static Encode encode;
}
```
通过`@DeclareParents`注解将 Encode 接口引入到 SpringIntroduction 接口的实现中。

`@DeclareParents` 注解的组成包括三点：
+ value属性用于匹配那些beans需要被引入这个新的接口。在这个例子中是所有SpringIntroduction的实现都被引入了新的接口（最后的那个+表示，所有SpringIntroduction的子类型，除了SpringIntroduction自己）。
+ defaultImpl属性用于指定一个新引入的接口的实现，在这里我们提供了DefaultEncode类；
+ 引入的新接口被定义为public static的属性，这里引入了Encode接口

### 使用
junit 的使用例子

```
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(loader = AnnotationConfigContextLoader.class)
public class EncodeIntroducerTest {
    @Autowired
    private SpringIntroduction springIntroduction;

    @Test
    public void testEncode() {
        Encode encode = (Encode) springIntroduction; //使用方法
        Encode.aopEncode();
    }
    
    @Bean
    public EncodeIntroducer encodeIntroducer() {
        return  new EncodeIntroducer();
    }
    
    @Configuration
    @EnableAspectJAutoProxy
    public static class EncdeIntroducerTest {
    }
}

```

## 附录 切入点表达式的例子

+ 任意公共方法的执行：
`execution(public * *(..))`

+ 任何一个名字以“set”开始的方法的执行：
`execution(* set*(..))`

+ AccountService接口定义的任意方法的执行：
`execution(* com.xyz.service.AccountService.*(..))`

+ 在service包中定义的任意方法的执行：
`execution(* com.xyz.service.*.*(..))`

+ 在service包或其子包中定义的任意方法的执行：
`execution(* com.xyz.service..*.*(..))`

+ 在service包中的任意连接点(在Spring AOP中只是方法执行)：
`within(com.xyz.service.*)`

+ 在service包或其子包中的任意连接点(在Spring AOP中只是方法执行)：
`within(com.xyz.service..*)`

+ 实现了AccountService接口的代理对象的任意连接点 (在Spring AOP中只是方法执行)：
`this(com.xyz.service.AccountService)`
'this'在绑定表单中更加常用：- 请参见后面的通知一节中了解如何使得代理对象在通知体内可用。

+ 实现AccountService接口的目标对象的任意连接点 (在Spring AOP中只是方法执行)：
`target(com.xyz.service.AccountService)`
'target'在绑定表单中更加常用：- 请参见后面的通知一节中了解如何使得目标对象在通知体内可用。

+ 任何一个只接受一个参数，并且运行时所传入的参数是Serializable 接口的连接点(在Spring AOP中只是方法执行)
`args(java.io.Serializable)`
'args'在绑定表单中更加常用：- 请参见后面的通知一节中了解如何使得方法参数在通知体内可用。
请注意在例子中给出的切入点不同于 execution(* *(java.io.Serializable))： args版本只有在动态运行时候传入参数是Serializable时才匹配，而execution版本在方法签名中声明只有一个 Serializable类型的参数时候匹配。

+ 目标对象中有一个 @Transactional 注解的任意连接点 (在Spring AOP中只是方法执行)
`@target(org.springframework.transaction.annotation.Transactional)`
'@target'在绑定表单中更加常用：- 请参见后面的通知一节中了解如何使得注解对象在通知体内可用。

+ 任何一个目标对象声明的类型有一个 @Transactional 注解的连接点 (在Spring AOP中只是方法执行)：
`@within(org.springframework.transaction.annotation.Transactional)`
'@within'在绑定表单中更加常用：- 请参见后面的通知一节中了解如何使得注解对象在通知体内可用。

+ 任何一个执行的方法有一个 @Transactional 注解的连接点 (在Spring AOP中只是方法执行)
`@annotation(org.springframework.transaction.annotation.Transactional)`
'@annotation'在绑定表单中更加常用：- 请参见后面的通知一节中了解如何使得注解对象在通知体内可用。

+ 任何一个只接受一个参数，并且运行时所传入的参数类型具有@Classified 注解的连接点(在Spring AOP中只是方法执行)
`@args(com.xyz.security.Classified)`
`@args`在绑定表单中更加常用：- 请参见后面的通知一节中了解如何使得注解对象在通知体内可用。

+ 任何一个在名为'tradeService'的Spring bean之上的连接点 (在Spring AOP中只是方法执行)：
`bean(tradeService)`
任何一个在名字匹配通配符表达式'*Service'的Spring bean之上的连接点 (在Spring AOP中只是方法执行)：
bean(*Service)



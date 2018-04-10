title: SpringTesting （一） Annotations
date: 2015-08-11 15:56:15
tags: [spring, junit, test]
categories: spring
---
[Annotations](http://docs.spring.io/spring/docs/4.2.1.BUILD-SNAPSHOT/spring-framework-reference/htmlsingle/#integration-testing-annotations)

##  Spring Testing Annotations


* ` @ContextConfiguration `
类级别注解，用来声明如何加载和配置`ApplicationContext`。可以使用XML配置文件和被`@Configuration`注解的类。
例如：
```
// 1） xml
@ContextConfiguration("/test-config.xml")
public class XmlApplicationContextTests {
    // class body...
}
  
// 2）  @Configuration Class
@ContextConfiguration(classes = TestConfig.class)
public class ConfigClassApplicationContextTests {
    // class body...
}
  
// 3）  `ApplicationContextInitializer` Class
@ContextConfiguration(initializers = CustomContextIntializer.class)
public class ContextInitializerTests {
    // class body...
}
  
// 4）  ContextLoader
@ContextConfiguration(locations = "/test-context.xml", loader = CustomContextLoader.class)
public class CustomLoaderXmlApplicationContextTests {
    // class body...
}
```

* @WebAppConfiguration
类级别注解，用来声明如何加载`WebApplicationContext`。默认`"file:src/main/webapp"`为web App根目录。必须协同` @ContextConfiguration`才生效
```
@ContextConfiguration
@WebAppConfiguration
public class WebAppTests {
    // class body...
}
    
    
    
    
@ContextConfiguration
@WebAppConfiguration("classpath:test-web-resources")
public class WebAppTests {
    // class body...
}
```


* @ContextHierarchy
类级别注解，声明多层级的`@ContextConfiguration`
```
@ContextHierarchy({
    @ContextConfiguration("/parent-config.xml"),
    @ContextConfiguration("/child-config.xml")
})
public class ContextHierarchyTests {
    // class body...
}
```
```
@WebAppConfiguration
@ContextHierarchy({
    @ContextConfiguration(classes = AppConfig.class),
    @ContextConfiguration(classes = WebConfig.class)
})
public class WebIntegrationTests {
    // class body...
}
```


* @ActiveProfiles
类级别注解，声明*ApplicationContext*使用哪些profiles。
``` 
@ContextConfiguration
@ActiveProfiles("dev")
public class DeveloperTests {
    // class body...
}
```
```
@ContextConfiguration
@ActiveProfiles({"dev", "integration"})
public class DeveloperIntegrationTests {
    // class body...
}
```


* @TestPropertySource
类级别注解，指定加载properties文件或手动增加PropertySources的内容
```
@ContextConfiguration
@TestPropertySource("/test.properties")
public class MyIntegrationTests {
    // class body...
}
```
```
@ContextConfiguration
@TestPropertySource(properties = { "timezone = GMT", "port: 4242" })
public class MyIntegrationTests {
    // class body...
}
```


* @DirtiesContext
类或方法级别注解，根据不同策略，Spring TestContext会刷新Spring的上下文（就是重新创建ApplicationContext）
有各种策略
```
// BEFORE_CLASS
@DirtiesContext(classMode = BEFORE_CLASS)
public class FreshContextTests {
    // some tests that require a new Spring container
}
  
  
  
//default class mode ： `AFTER_CLASS`
@DirtiesContext
public class ContextDirtyingTests {
    // some tests that result in the Spring container being dirtied
}
  
  
  
// BEFORE_EACH_TEST_METHOD
@DirtiesContext(classMode = BEFORE_EACH_TEST_METHOD)
public class FreshContextTests {
    // some tests that require a new Spring container
}
  
  
  
// AFTER_EACH_TEST_METHOD
@DirtiesContext(classMode = AFTER_EACH_TEST_METHOD)
public class ContextDirtyingTests {
    // some tests that result in the Spring container being dirtied
}
  
  
  
// BEFORE_METHOD
@DirtiesContext(methodMode = BEFORE_METHOD)
@Test
public void testProcessWhichRequiresFreshAppCtx() {
    // some logic that requires a new Spring container
}
  
  
  
//default method mode `AFTER_METHOD`
@DirtiesContext
@Test
public void testProcessWhichDirtiesAppCtx() {
    // some logic that results in the Spring container being dirtied
} 
```

* `@TestExecutionListeners`
* `@Commit`
* `@Rollback`
* `@BeforeTransaction`
* `@AfterTransaction`
* `@Sql`
* `@SqlConfig`
* `@SqlGroup`



## Standard Annotation Support
Spring TestContext Framework 支持下列标准的注解
*     @Autowired
*    @Qualifier
*    @Resource (javax.annotation) if JSR-250 is present
*    @Inject (javax.inject) if JSR-330 is present
*    @Named (javax.inject) if JSR-330 is present
*    @PersistenceContext (javax.persistence) if JPA is present
*    @PersistenceUnit (javax.persistence) if JPA is present
*    @Required
*    @Transactional 


## Spring JUnit Testing Annotations
组合SpringJUnit4ClassRunner, Spring’s JUnit rules, 或 Spring’s JUnit support classes使用的时候，Spring TestContext Framework 还支持下列标准的注解：
* @IfProfileValue
类级别或方法级别注解，校验具体的环境变量，匹配才进行测试，否则就会忽略。
```
@IfProfileValue(name="java.vendor", value="Oracle Corporation")
@Test
public void testProcessWhichRunsOnlyOnOracleJvm() {
    // some logic that should run only on Java VMs from Oracle Corporation
}
  
  
@IfProfileValue(name="test-groups", values={"unit-tests", "integration-tests"})
@Test
public void testProcessWhichRunsForUnitOrIntegrationTestGroups() {
    // some logic that should run only for unit and integration test groups
}
```

* @ProfileValueSourceConfiguration

* @Timed
```
@Timed(millis=1000)
public void testProcessWithOneSecondTimeout() {
    // some logic that should not take longer than 1 second to execute
}
```

* @Repeat
```
@Repeat(10)
@Test
public void testProcessRepeatedly() {
    // ...
}
```

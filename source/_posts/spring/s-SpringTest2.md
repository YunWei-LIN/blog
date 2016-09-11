title: SpringTesting （二） Spring TestContext Framework
date: 2015-08-12 19:56:15
tags: Spring Testing
categories: spring
---
[Spring TestContext Framework](http://docs.spring.io/spring/docs/4.2.1.BUILD-SNAPSHOT/spring-framework-reference/htmlsingle/#testcontext-framework)

## TODO 设计原理和实现

` TestContextManager` ： Spring TestContext Framework的主入口，管理单例的`TestContext`和所有注册的TestExecutionListeners。
` TestContext`
` TestExecutionListener`
` ContextLoader`



## Context management
每个TestContext提供 context 管理， 测试实例无权配置ApplicationContext， 实例仅能获得一个ApplicationContext的引用。特别的是AbstractJUnit4SpringContextTests和AbstractTestNGSpringContextTests实现了ApplicationContextAware而且有权修改ApplicationContext。
```
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration
public class MyTest {

    @Autowired
    private ApplicationContext applicationContext;

    // class body...
}
  
  
@RunWith(SpringJUnit4ClassRunner.class)
@WebAppConfiguration
@ContextConfiguration
public class MyWebAppTest {
    @Autowired
    private WebApplicationContext wac;

    // class body...
}
```


## 注入mocks
```
@WebAppConfiguration
@ContextConfiguration
public class WacTests {

    @Autowired
    WebApplicationContext wac; // cached

    @Autowired
    MockServletContext servletContext; // cached

    @Autowired
    MockHttpSession session;

    @Autowired
    MockHttpServletRequest request;

    @Autowired
    MockHttpServletResponse response;

    @Autowired
    ServletWebRequest webRequest;

    //...
}
```

## 执行SQL scripts
* 代码
下列都可以用来执行
     org.springframework.jdbc.datasource.init.ScriptUtils
    org.springframework.jdbc.datasource.init.ResourceDatabasePopulator
    org.springframework.test.context.junit4.AbstractTransactionalJUnit4SpringContextTests
    org.springframework.test.context.testng.AbstractTransactionalTestNGSpringContextTests 
```
@Test
public void databaseTest {
    ResourceDatabasePopulator populator = new ResourceDatabasePopulator();
    populator.addScripts(
        new ClassPathResource("test-schema.sql"),
        new ClassPathResource("test-data.sql"));
    populator.setSeparator("@@");
    populator.execute(this.dataSource);
    // execute code that uses the test schema and data
}
```

* @Sql
```
//1
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration
@Sql("/test-schema.sql")
public class DatabaseTests {

    @Test
    public void emptySchemaTest {
        // execute code that uses the test schema without any test data
    }

    @Test
    @Sql({"/test-schema.sql", "/test-user-data.sql"})
    public void userTest {
        // execute code that uses the test schema and test data
    }
}
  
  

//1
@Test
@Sql(scripts = "/test-schema.sql", config = @SqlConfig(commentPrefix = "`"))
@Sql("/test-user-data.sql")
public void userTest {
    // execute code that uses the test schema and test data
}
  
  
//3
@Test
@SqlGroup({
    @Sql(scripts = "/test-schema.sql", config = @SqlConfig(commentPrefix = "`")),
    @Sql("/test-user-data.sql")
)}
public void userTest {
    // execute code that uses the test schema and test data
}
  
  
//4
@Test
@Sql(
    scripts = "create-test-data.sql",
    config = @SqlConfig(transactionMode = ISOLATED)
)
@Sql(
    scripts = "delete-test-data.sql",
    config = @SqlConfig(transactionMode = ISOLATED),
    executionPhase = AFTER_TEST_METHOD
)
public void userTest {
    // execute code that needs the test data to be committed
    // to the database outside of the test's transaction
}
```


## TestContext Framework支持类
### Spring JUnit Runner
```
@RunWith(SpringJUnit4ClassRunner.class)
@TestExecutionListeners({})
public class SimpleTest {

   @Test
   public void testMethod() {
      // execute test logic...
   }
}
```
如果要使用MockitoJUnitRunner等第三方runners， 可以使用Spring JUnit Rules。

### Spring JUnit Rules
```
// Optionally specify a non-Spring Runner via @RunWith(...)
@ContextConfiguration
public class IntegrationTest {

   @ClassRule
   public static final SpringClassRule SPRING_CLASS_RULE = new SpringClassRule();

   @Rule
   public final SpringMethodRule springMethodRule = new SpringMethodRule();

   @Test
   public void testMethod() {
      // execute test logic...
   }
}
```
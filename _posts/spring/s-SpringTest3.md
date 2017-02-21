title: SpringTesting （三） Spring MVC Test Framework
date: 2015-08-13 19:56:15
tags: Spring Testing
categories: spring
---
[Spring MVC Test Framework](http://docs.spring.io/spring/docs/4.2.1.BUILD-SNAPSHOT/spring-framework-reference/htmlsingle/#spring-mvc-test-framework)建立在Servlet API mock objects基础上，他不需要一个运行的Servlet容器， 
不需要，不需要，不需要！
他使用DispatcherServlet来提供完整Spring MVC的支持，使用TestContext framework来加载实际的Spring各个配置。

## Server-Side Tests
 Spring MVC Test的目的：提供一种有效的利用DispatcherServlet所伴生的requests和responses来测试controller的方式。
 例如
```
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

@RunWith(SpringJUnit4ClassRunner.class)
@WebAppConfiguration
@ContextConfiguration("test-servlet-context.xml")
public class ExampleTests {

    @Autowired
    private WebApplicationContext wac;

    private MockMvc mockMvc;

    @Before
    public void setup() {
        this.mockMvc = MockMvcBuilders.webAppContextSetup(this.wac).build();
    }

    @Test
    public void getAccount() throws Exception {
        this.mockMvc.perform(get("/accounts/1").accept(MediaType.parseMediaType("application/json;charset=UTF-8")))
            .andExpect(status().isOk())
            .andExpect(content().contentType("application/json"))
            .andExpect(jsonPath("$.name").value("Lee"));
    }

}
```


### Setup Options
* webAppContextSetup
```
@RunWith(SpringJUnit4ClassRunner.class)
@WebAppConfiguration
@ContextConfiguration("my-servlet-context.xml")
public class MyWebTests {

    @Autowired
    private WebApplicationContext wac;

    private MockMvc mockMvc;

    @Before
    public void setup() {
        this.mockMvc = MockMvcBuilders.webAppContextSetup(this.wac).build();
    }

    // ...

}
```

* standaloneSetup（简单controller实例）
```
public class MyWebTests {

    private MockMvc mockMvc;

    @Before
    public void setup() {
        this.mockMvc = MockMvcBuilders.standaloneSetup(new AccountController()).build();
    }

    // ...

}
```

如何选择？
webAppContextSetup 方式：加载并缓存完整的Spring MVC配置，

standaloneSetup    方式：更像一个单元测试

他们就像集成测试Vs单元测试。


### Performing Requests
```
mockMvc.perform(post("/hotels/{id}", 42).accept(MediaType.APPLICATION_JSON));
  
mockMvc.perform(fileUpload("/doc").file("a1", "ABC".getBytes("UTF-8")));
  
mockMvc.perform(get("/hotels?foo={foo}", "bar"));
  
mockMvc.perform(get("/hotels").param("foo", "bar"));
  
mockMvc.perform(get("/app/main/hotels/{id}").contextPath("/app").servletPath("/main"))
```

### 测试预期
测试预期结果可以在Requests后面加一个或多个` .andExpect(..)`
```
mockMvc.perform(post("/persons"))
    .andDo(print())  // static import from MockMvcResultHandlers， can print all the available result data
    .andExpect(status().isOk())
    .andExpect(model().attributeHasErrors("person"));
```

如果需要直接访问结果来验证一些其他数据，在测试预期最后加上` .andReturn()`
```
MvcResult mvcResult = mockMvc.perform(post("/persons")).andExpect(status().isOk()).andReturn();
```

### [HtmlUnit Integration TODO](http://docs.spring.io/spring/docs/4.2.1.BUILD-SNAPSHOT/spring-framework-reference/htmlsingle/#spring-mvc-test-server-htmlunit)


## Client-Side REST Tests
使用RestTemplate进行客户端的REST测试。
```
RestTemplate restTemplate = new RestTemplate();

MockRestServiceServer mockServer = MockRestServiceServer.createServer(restTemplate);
mockServer.expect(requestTo("/greeting")).andRespond(withSuccess("Hello world", MediaType.TEXT_PLAIN));

// use RestTemplate ...

mockServer.verify();
```


## PetClinic Example
Spring官方提供的完整案例[Github](https://github.com/spring-projects/spring-petclinic)
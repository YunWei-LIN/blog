---
title: springfox 整合springmvc 生成rest接口文档
date: 2017-03-14 18:31:21
tags: [spring, springfox]
categories: spring
---

主要内容


`springfox` 可以用来快速生成RESTful API文档,使后台开发人员与移动端开发人员更好的对接.
前身是 `swagger-springmvc` , 根基都是 `swagger`

本文以 `springfox-swagger2` 版本为例说明。

<!-- more -->

## Maven 依赖
```xml
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.4.0</version>
</dependency>
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.4.0</version>
</dependency>
```

## swagger2 配置文件
```java
@Configuration
@EnableSwagger2
public class SwaggerConfig {

    @Bean
    public Docket buildDocket(){
        return new Docket(DocumentationType.SWAGGER_2)
                .host("localhost:8080") //服务地址和端口
                .apiInfo(buildApiInf())
                .select()
                .apis(RequestHandlerSelectors.basePackage("com.springfox.controller"))//controller路径
                .paths(PathSelectors.any())
                .build();
    }

    private ApiInfo buildApiInf(){
        return new ApiInfoBuilder()
                .title("springfox 大标题")
                .termsOfServiceUrl("http://giveme5.cc/")
                .description("springmvc swagger2")
                .contact(new Contact("sam", "http://giveme5.cc/", "xxxxxxxxxxxx@163.com"))
                .build();

    }
}
```

## SpringMvc 配置

+ java based config
  
    ```java
    @Configuration
    @EnableWebMvc
    @EnableSpringDataWebSupport
    @Import(
    { SwaggerConfig.class})
    public class ApplicationConfiguration extends WebMvcConfigurerAdapter
    {
    ...

        // Maps resources path to webapp/resources
        @Override
        public void addResourceHandlers(ResourceHandlerRegistry registry)
        {
            registry.addResourceHandler("swagger-ui.html").addResourceLocations(
                    "classpath:/META-INF/resources/");

            registry.addResourceHandler("/webjars/**").addResourceLocations("classpath:/META" +
                    "-INF/resources/webjars/");


        }

    ...
    }
    ```
  
+ xml based config
  
    ```xml
    <!--<bean class="com.springfox.config.SwaggerConfig" /> 使用bean申明可以去掉@configuration-->
    <!--扫描@configuration注解-->
    <context:component-scan base-package="com.springfox.config"/>
    <!--配置静态资源访问-->
    <mvc:resources mapping="swagger-ui.html" location="classpath:/META-INF/resources/"/>
    <mvc:resources mapping="/webjars/**" location="classpath:/META-INF/resources/webjars/"/>
    ```

## 接口举例
```java
@RestController
@RequestMapping("/api/appoint/")
@Api(value = "appoint",description = "预约 Controller", tags = {"医疗-预约"})
public class AppointAppController {

    @Autowired
    private IAptAppointService appointService;

    @Autowired
    private Environment env;

    /**
     * logger
     */
    private final Logger logger = LoggerFactory.getLogger(AppointAppController.class);


    /**
     * 加载预约排班信息
     *
     * @return
     */
    @ApiImplicitParams({
            @ApiImplicitParam(name = "doctCode", value = "医生code"),
            @ApiImplicitParam(name = "deptCode", value = "科室code")
    })
    @ApiOperation("预约排班信息")
    @RequestMapping(value = "loadSchedule", method = RequestMethod.POST)
    public ComServiceResVo loadSchedule(@RequestParam String doctCode, @RequestParam String deptCode) {

        。。。
        
        return ComServiceResVo.returnList(list);
    }
}
```

+ @Api:标志这个类为Swagger资源
+ @ApiImplicitParam:对单个参数进行说明,其中dataType一定为小写
+ @ApiOperation:描述了一种操作或通常针对特定的路径的HTTP方法。

>[更多注解 官方说明](https://github.com/swagger-api/swagger-core/wiki/Annotations-1.5.X)

## 最终结果
打开 [http://localhost:8080/project_name/swagger-ui.html](http://localhost:8080/swagger-ui.html#) ,project_name表示你启动项目的名称,如果你以根目录启动则没有project_name,当你看到如下界面就表示配置成功了
![](/images/swagger.png)

点开具体接口， 有惊喜 哈哈。
`Try it out！` 可以实时测试接口

>[完整官方文档](https://springfox.github.io/springfox/docs/snapshot/#getting-started)

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
    <version>2.6.1</version>
</dependency>
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.6.1</version>
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

+ controller
    ```java
    @RestController
    @RequestMapping("/api/appoint/")
    @Api(value = "appoint",description = "预约 模块")
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
        @ApiOperation(value = "预约排班信息",  notes="获取医生排班信息")
        @RequestMapping(value = "loadSchedule", method = RequestMethod.POST)
        public ComResultVo<UserEntity> loadSchedule(@RequestParam String doctCode, @RequestParam String deptCode) {

            。。。
            
            return ComResultVo.returnList(list);
        }
    }
    ```

+ entity
UserEntity
    ```java
    package com.xxxx.domain;

    import io.swagger.annotations.ApiModel;
    import io.swagger.annotations.ApiModelProperty;

    import javax.persistence.*;
    import javax.validation.constraints.NotNull;

    @Entity
    @ApiModel()
    public class UserEntity extends BaseEntity {

        @Id
        @TableGenerator(
                name = "User_gen",
                table = "t_com_id_generator_r",
                pkColumnName = "seq_name",
                pkColumnValue = "User_id",
                valueColumnName = "seq_value",
                allocationSize = 50
        )
        @GeneratedValue(
                strategy = GenerationType.TABLE,
                generator = "User_gen"
        )
        @ApiModelProperty(value = "用户ID", notes = "user's name xxxxxxxxxxxxxx")
        private Long id;


        /**
        * 用户名称
        */
        @ApiModelProperty(value = "用户姓名", notes = "user's name xxxxxxxxxxxxxx")
        @NotNull
        private String name;

        public Long getId() {
            return id;
        }

        public void setId(Long id) {
            this.id = id;
        }

        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }
    }
    ```
    ComResultVo
    ```java
    package com.xxx.domain;

    import io.swagger.annotations.ApiModel;
    import io.swagger.annotations.ApiModelProperty;
    import org.springframework.data.domain.Page;

    import java.io.Serializable;
    import java.util.List;

    @ApiModel()
    public class ComResultVo<T> implements Serializable {

            private static final long serialVersionUID = 176607880345671118L;
            public static final int COUNT_ZERO = 0;
            private boolean result = false;

            @ApiModelProperty(value = "message ", notes = "user's name xxxxxxxxxxxxxx")
            private String message;
            private int status;
            private long count = 0L;
            private T entity;
            private List<T> list;

            public ComResultVo() {
            }

            public ComResultVo returnEntity(T entity) {
                return createComResutVo(true, ComResultVo.Status.SUCCESS.getCode(), (String)null, (Long)null, entity, (List)null);
            }

            public ComResultVo returnList(List<T> data) {
                return returnPagedList(data, data == null?null:Long.valueOf((long)data.size()));
            }

            public ComResultVo returnPagedList(List<T> data, Long count) {
                return count != null && count.intValue() != 0?createComResutVo(true, ComResultVo.Status.SUCCESS.getCode(), (String)null, count, null, data):createComResutVo(true, ComResultVo.Status.SUCCESS.getCode(), "暂时没有相关的信息喔", count, null, data);
            }

            public ComResultVo returnPages(Page<T> data) {
                return data.getTotalElements() == 0L?
                        createComResutVo(true, ComResultVo.Status.SUCCESS.getCode(), "暂时没有相关的信息喔", Long.valueOf(data.getTotalElements()), null, data.getContent()):
                        createComResutVo(true, ComResultVo.Status.SUCCESS.getCode(), (String)null, Long.valueOf(data.getTotalElements()), null, data.getContent());
            }

            public ComResultVo returnSuccessStatus() {
                return createComResutVo(true, ComResultVo.Status.SUCCESS.getCode(), (String)null, (Long)null, null, (List)null);
            }

            public ComResultVo returnErrorMessage(String message) {
                return createComResutVo(false, ComResultVo.Status.BAD_REQUEST.getCode(), message, (Long)null, null, null);
            }

            public ComResultVo returnStatus(boolean result, int status, String message) {
                return createComResutVo(result, status, message, (Long)null, null, null);
            }

            public ComResultVo returnALL(boolean result, int status, String message, long count, T entity, List<T> list) {
                return createComResutVo(result, status, message, Long.valueOf(count), entity, list);
            }

            private ComResultVo createComResutVo(boolean result, int status, String message, Long count, T entity, List<T> list) {
                ComResultVo vo = new ComResultVo();
                vo.result = result;
                vo.status = status;
                if(message == null) {
                    if(result) {
                        vo.message = ComResultVo.Status.SUCCESS.getDescription();
                    } else {
                        vo.message = "操作失败";
                    }
                } else {
                    vo.message = message;
                }

                if(count == null) {
                    if(null != list) {
                        vo.count = (long)list.size();
                    } else {
                        vo.count = 0L;
                    }
                } else {
                    vo.count = count.longValue();
                }

                vo.entity = entity;
                vo.list = list;
                return vo;
            }

            public boolean isResult() {
                return this.result;
            }

            public void setResult(boolean result) {
                this.result = result;
            }

            public String getMessage() {
                return this.message;
            }

            public void setMessage(String message) {
                this.message = message;
            }

            public int getStatus() {
                return this.status;
            }

            public void setStatus(int status) {
                this.status = status;
            }

            public long getCount() {
                return this.count;
            }

            public void setCount(long count) {
                this.count = count;
            }

            public T getEntity() {
                return this.entity;
            }

            public void setEntity(T entity) {
                this.entity = entity;
            }

            public List<T> getList() {
                return this.list;
            }

            public void setList(List<T> list) {
                this.list = list;
            }
    }
    ```

+ @Api:标志这个类为Swagger资源
+ @ApiImplicitParam:对单个参数进行说明,其中dataType一定为小写
+ @ApiOperation:描述了一种操作或通常针对特定的路径的HTTP方法。
+ @ApiModel: 描述一个实体
+ @ApiModelProperty：描述一个字段

>[更多注解 官方说明](https://github.com/swagger-api/swagger-core/wiki/Annotations-1.5.X)

### 泛型返回值 response
`ComResultVo<UserEntity>` 这种带泛型的返回值，想要正常完全显示，需要注意如下：
+ `ComResultVo` 内需要正确的 `get/set` 方法
+ controller 返回值 `ComResultVo<UserEntity>` 一定要 标明泛型的具体值 `UserEntity`

达到如下效果
![](/images/springfox.png)

## 最终结果
打开 [http://localhost:8080/project_name/swagger-ui.html](http://localhost:8080/swagger-ui.html#) ,project_name表示你启动项目的名称,如果你以根目录启动则没有project_name,当你看到如下界面就表示配置成功了
![](/images/swagger.png)

点开具体接口， 有惊喜 哈哈。
`Try it out！` 可以实时测试接口

>[完整官方文档](https://springfox.github.io/springfox/docs/snapshot/#getting-started)

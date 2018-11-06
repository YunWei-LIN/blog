---
title: swagger springfox rest接口文档
date: 2018-11-04 18:31:21
tags: [spring, springfox, swagger]
categories: spring
---

主要内容


`swagger`(`springfox`) 可帮助开发人员设计，构建，记录和使用RESTful Web服务, 使后台开发人员与移动端开发人员更好的对接.
大多数用户通过Swagger UI工具可很简单识别和使用Swagger。

最大优点： 接口开发人员不用另外写接口文档，代码注释中写上swagger相关的注释就可以自动生成接口文档；
最大缺点： 对源代码侵入比较严重。



本文以 `springfox-swagger2` V2.9.2 版本为例说明。

末了，还有个稍重量级竞品 [RAP(阿里妈妈出品) ](https://github.com/thx/rap2-delos) ， 感兴趣的可以去玩玩。

*更新历史*
+2018-11-04: 增加swagger注解具体说明

<!-- more -->

## Maven 依赖
```xml
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.9.2</version>
</dependency>
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.9.2</version>
</dependency>
```

## swagger2 配置文件
```java
@Configuration
@EnableSwagger2
@ConditionalOnExpression("${giveme5.swagger.enable:true}") //是否启用Swagger的判断
public class SwaggerConfig {

    /**
     * 模拟接口的认证授权
     * 配合 @ApiOperation(authorizations = { @Authorization(value="apiKey") })
     * @return
     */
    private ApiKey apiKey() {
        return new ApiKey("apiKey", "Authorization", "header");
    }


    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .securitySchemes(Arrays.asList(apiKey()))
                .select()
                    .apis(RequestHandlerSelectors.withClassAnnotation(Api.class))
                    .paths(PathSelectors.ant("/app/**")) // 只对 url 匹配 /app/** 的生效
                    .build()


                ;
    }

    private ApiInfo apiInfo(){
        return new ApiInfoBuilder()
                .title("springfox 大标题")
                .termsOfServiceUrl("http://giveme5.cc/")
                .description("springmvc swagger2")
                .contact(new Contact("sam", "http://giveme5.cc/", "xxxxxxxxxxxx@163.com"))
                .build();

    }
}
```

## Spring 配置

### SpringBoot
Springboot 可以直接使用

### SpringMvc 配置

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
    
/**
 * 类功能说明：注册用户 controller
 */

@RestController
@RequestMapping("/app")
@Api(tags = {"用户接口"})
public class CustomerAppController {

    /**
     * 第三方登录 - 刷新
     *
     * @param token     QQ授权后的access_token 或 微信的 refresh_token 或 微博的 refresh_token
     * @param loginType 1: QQ； 2: 微信； 3：微博
     * @param phone 手机号码
     * @param valid 校验码
     * @return
     */
    @PostMapping(value = "/s/login/refresh")
    @ApiOperation(value = "第三方登录 - 刷新", response = ThirdLoginToken.class,
            notes = "刷新成功后 返回 Map {jwt: jwt token, accessToken, refreshToken} \n 成功后 在http header中添加 域: Authorization=jwt的值 ")
    @ApiImplicitParams({
        @ApiImplicitParam(name = "token",  required = true, value = "授权后 QQ的 或 微信的 或 微博的 refresh_token"),
        @ApiImplicitParam(name = "loginType", allowableValues = "1,2,3", dataType = "int", required = true, value = "第三方登陆类型  1: QQ； 2: 微信； 3：微博"),
        @ApiImplicitParam(name = "phone", dataType = "int", value = "初次授权，需要绑定的 手机"),
        @ApiImplicitParam(name = "valid", dataType = "int", value = "初次授权，需要绑定的 手机校验码")
    })
    public ResponseEntity<ThirdLoginToken> refresh(
            @RequestParam String token,
            @RequestParam int loginType,
            @RequestParam(required = false) String phone,
            @RequestParam(required = false) String valid
        ) throws IOException {

        return thirdLogin(token, loginType, LOGIN_REFRESH.REFRESH.ordinal(), phone, valid);
    }

    /**
     * 获取用户信息
     *
     * @return
     */
    @GetMapping(value = "/customer")
    @ApiOperation(value = "获取用户信息", authorizations = { @Authorization(value="apiKey") }, response = CustomerEntity.class)
    public ResponseEntity<CustomerEntity> getCustomer(){
        。。。

        return ResponseEntity.ok(customer);
    }

    /**
     * 绑定 qq, 微信，微博等
     *
     * @param token
     * @param loginType
     * @return
     */
    @PutMapping(value = "/customer/third")
    @ApiOperation(value = "绑定 qq, 微信，微博等 ", response = ThirdLoginToken.class, authorizations = { @Authorization(value="apiKey") })
    @ApiImplicitParams({
            @ApiImplicitParam(name = "token",  required = true, value = "QQ授权后的access_token 或 微信的 authorization_code 或 微博的 authorization_code"),
            @ApiImplicitParam(name = "loginType", allowableValues = "1,2,3", dataType = "int", required = true, value = "第三方登陆类型  1: QQ； 2: 微信； 3：微博")
    })
    public ResponseEntity updateCustomerThirdLogin(@RequestParam String token, @RequestParam int loginType) throws IOException {
        ThirdLoginToken loginToken = new ThirdLoginToken();
        ...


        return ResponseEntity.ok(loginToken);
    }





}

    ```

+ entity

    ```java
    /**
     * 第三方登陆返回 VO
     */
    @ApiModel
    private class ThirdLoginToken {

        @ApiModelProperty(value = "jwt token, 获取后 在http header中添加 域: Authorization=jwt的值")
        public String jwt;

        @ApiModelProperty(value = "第三方授权的 access token")
        public String accessToken;

        @ApiModelProperty(value = "第三方授权的 refresh token")
        public String refreshToken;

        @ApiModelProperty(value = "第三方授权的 openId")
        public String openId;

        public ThirdLoginToken(String jwt, String accessToken, String refreshToken, String openId) {
            this.jwt = jwt;
            this.accessToken = accessToken;
            this.refreshToken = refreshToken;
            this.openId = openId;
        }
    }
    ```

+ @Api:标志这个类为Swagger资源，根据config， 没标注的不会生成 swagger 的接口文档
+ @ApiImplicitParams，@ApiImplicitParam: 对参数进行说明, 其中dataType一定为小写； allowableValues 可限制 合法值得列表； required 指定该参数是否必须
+ @ApiOperation:描述了一种操作或通常针对特定的路径的HTTP方法。 response 指定返回值类型；authorizations 指定改接口的认证条件， apiKey 需要和 config的一致；
+ @ApiModel: 描述一个实体
+ @ApiModelProperty：描述一个字段

>[更多注解 官方说明](https://github.com/swagger-api/swagger-core/wiki/Annotations-1.5.X)

### 泛型返回值 response
`ComResultVo<UserEntity>` 这种带泛型的返回值，想要正常完全显示，需要注意如下：
+ `ComResultVo` 内需要正确的 `get/set` 方法
+ controller 返回值 `ComResultVo<UserEntity>` 一定要 标明泛型的具体值 `UserEntity`

达到如下效果
![](/images/swagger_model.png)
![](/images/swagger_model2.png)

### 接口认证
`Authorize` 按钮， 填入合法的认证值，模拟授权。

## 最终结果
打开 [http://localhost:8080/project_name/swagger-ui.html](http://localhost:8080/swagger-ui.html#) ,project_name表示你启动项目的名称,如果你以根目录启动则没有project_name,当你看到如下界面就表示配置成功了
![](/images/swagger.png)

点开具体接口， 有惊喜 哈哈。
`Try it out！` 可以实时测试接口

>[完整官方文档](https://springfox.github.io/springfox/docs/snapshot/#getting-started)

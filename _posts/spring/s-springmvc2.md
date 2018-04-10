title: Spring MVC（二） 配置
date: 2015-08-07 13:17:37
tags: [spring, mvc]
categories: spring
---

## Spring MVC 的配置 [Configuring Spring MVC](http://docs.spring.io/spring/docs/4.2.1.BUILD-SNAPSHOT/spring-framework-reference/htmlsingle/#mvc-config)



### * 开启Spring MVC默认配置
Java Config: 加入` @EnableWebMvc` 到 某个有 ` @Configuration ` 注解的类头。

```
@Configuration
@EnableWebMvc
public class WebConfig {

}
```

XML方式: 增加` mvc:annotation-driven ` 到 DispatcherServlet 的context里面。
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:mvc="http://www.springframework.org/schema/mvc"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/mvc
        http://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <mvc:annotation-driven />

</beans>
```


### * 自定义Spring MVC配置

Java Config: 加入` @EnableWebMvc` 到 某个有 ` @Configuration ` 注解的类头。自己实现` WebMvcConfigurer`， 更多是继承` WebMvcConfigurerAdapter`并重写你自己需要的方法。

```
@Configuration
@EnableWebMvc
public class WebConfig extends WebMvcConfigurerAdapter {

    // Override configuration methods...

}
```

XML方式: 增加` mvc:annotation-driven ` 到 DispatcherServlet 的context里面。设置自己需要[Spring MVC XML schema](http://schema.spring.io/mvc/spring-mvc.xsd)里面的元素值。




### * 高级自定义Spring MVC配置

Java Config: 去掉` @EnableWebMvc`。 ` @Configuration ` 注解类头，自己实现` WebMvcConfigurationSupport.`， 更多是继承` DelegatingWebMvcConfiguration`并重写你自己需要的方法。
``` 
@Configuration
public class WebConfig extends DelegatingWebMvcConfiguration {

    @Override
    public void addInterceptors(InterceptorRegistry registry){
        // ...
    }

    @Override
    @Bean
    public RequestMappingHandlerAdapter requestMappingHandlerAdapter() {
        // Create or let "super" create the adapter
        // Then customize one of its properties
    }

}
```


具体内容看头部的官方文档

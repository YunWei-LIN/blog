---
title: spring boot war包
date: 2018-11-21 18:31:21
tags: [spring, spring boot, war]
categories: spring
---

主要内容

spring boot 构建war包


本文以 `spring boot` V2.1.0 版本为例说明。



*更新历史*
无

---

<!-- more -->

`spring boot` 默认是 可执行的 JAR 包，如果需要构建 war包， 仅需修改如下2个文件

+ POM.xml
在原有基础下 增加 
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-tomcat</artifactId>
    <scope>provided</scope>
</dependency>
```


+ Application.java
继承 `SpringBootServletInitializer`， 实现方法 `configure(SpringApplicationBuilder application)`
类似如下：
```java

...
@SpringBootApplication
@EnableJpaAuditing
@EnableConfigurationProperties
public class Application extends SpringBootServletInitializer {

    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
        return application.sources(Application.class);
    }

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);

    }


...

}
```

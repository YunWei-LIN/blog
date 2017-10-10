title: spring 多态 注入
date: 2017-10-10 19:44:58
tags: [spring, 多态, 注入, DI]
categories: spring
---

## 主要内容
spring 多态 注入
利用 `PropertySource` 实现 多态的动态注入。

*更新历史*
无

<!-- more -->

---------------------------------------------------------------------

环境：spring 4, JDK 1.8； 话不多说，看代码

## 多态 
一个 service 接口， 有若干具体实现

### 接口 `IPolyDiService.java`
```java
package com.xxx.service;

public interface IPolyDiService {
    void printName();

}
```

### 实现A `APolyDiServiceImpl`
```java
package com.xxx.service.impl;

import com.xxx.service.IPolyDiService;
import org.springframework.stereotype.Service;

@Service("A_service")
public class APolyDiServiceImpl implements IPolyDiService {

    @Override
    public void printName() {
        System.out.println("This is A PolyDiServiceImpl");
    }
}
```

### 实现B `BPolyDiServiceImpl`
```java
package com.xxx.service.impl;

import com.xxx.service.IPolyDiService;
import org.springframework.stereotype.Service;

@Service("B_service")
public abstract class BPolyDiServiceImpl implements IPolyDiService {
    @Override
    public void printName() {
        System.out.println("This is B PolyDiServiceImpl");
    }
}
```

## 动态注入

### 配置文件 `env.properties`
可以用其他外置配置替代， 如下， 改变 `service` 的值， 可以运行时动态注入不同的具体实现。
```sh
# service=B
service=A
```

### Junit `PolyDiTest`
```java
package com.xxx.service;

import com.xxx.service.IPolyDiService;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.PropertySource;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
import org.springframework.test.context.support.AnnotationConfigContextLoader;

import javax.annotation.Resource;
import java.text.ParseException;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(loader = AnnotationConfigContextLoader.class)

public class PolyDiTest {

    @Resource(name = "${service}_service")
    private IPolyDiService service;

    @Test
    public void test() throws ParseException {
        service.printName();

    }

    @Configuration
    @PropertySource({ "classpath:conf/env.properties"})
    @ComponentScan(basePackages ={ "com.xxx.service.impl"})
    public static class ContextConfiguration {

    }
}

```

### 测试结果
`service=A` 的结果
```
This is A PolyDiServiceImpl
```

`service=B` 的结果
```
This is B PolyDiServiceImpl
```

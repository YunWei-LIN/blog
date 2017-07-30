---
title: Java 8 新特性 概要
date: 2017-06-30 22:18:03
tags: [java 8, jdk8]
categories: java
---

___主要内容___

Java 8是Java自Java 5之后的最重要的版本。这个版本包含语言、编译器、库、工具和JVM等方面的十多个新特性。
外面已经有很多资料， 我在这里仅仅做概要记录，提醒自己多使用新特性， 提高效率和性能， 详细内容参考附录。

<!-- more -->

## 语言的新特性

### 接口的默认方法和静态方法
java 8 的 接口， 不再是 老程序员 脑海中的接口了， java 8 允许在接口中有具体的实现： `默认方法`和`静态方法`

+ 默认方法
使用关键字 `default` 定义, 已经是具体的实现， 不需要实现类去现象， 可以被继承或者覆写


### Lambda表达式和函数式接口
java 的 Lambda 表达式借助`函数接口`实现，可以替换以前的匿名内部类。

+ 函数接口 
`@FunctionalInterface` 注解 `函数接口`， 函数接口指的是只有一个函数的接口，这样的接口可以隐式转换为Lambda表达式。
默认方法和静态方法不会破坏函数式接口的定义。

+ Lambda
有了 Lambda， 你就可以把一段代码(函数) 当做参数 传给 另一个函数。
可以去掉以前大量的匿名内部类。


### 重复注解
同一个地方多次使用同一个注解。

注解几乎可以使用在任何元素上：局部变量、接口类型、超类和接口实现类，甚至可以用在函数的异常定义上。

## 编译器
### 参数名称
默认关闭， 需要带 `-parameters` 参数编译

Maven 的使用
```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.1</version>
    <configuration>
        <compilerArgument>-parameters</compilerArgument>
        <source>1.8</source>
        <target>1.8</target>
    </configuration>
</plugin>
```

## 官方库
### Optional
Optional是一个容器：存放T类型的值或者null。它提供了一些有用的接口来避免显式的null检查

### Streams
Stream API（java.util.stream）将函数式编程引入了Java库中。
这是目前为止最大的一次对Java库的完善！！！！
极大得简化了集合操作

[java.util.stream](http://docs.oracle.com/javase/8/docs/api/java/util/stream/package-summary.html#StreamOps)
[使用Stream API处理集合](https://wizardforcel.gitbooks.io/java8-tutorials/content/Java%208%20%E6%96%B0%E7%89%B9%E6%80%A7%E4%B9%8B%E6%97%85%20%E4%BD%BF%E7%94%A8%20Stream%20API%20%E5%A4%84%E7%90%86%E9%9B%86%E5%90%88.html)

### Date/Time API(JSR 310)
新的时间和日期管理API
[java.time](http://docs.oracle.com/javase/8/docs/api/java/time/package-summary.html)

### Nashorn JavaScript引擎

### Base64
`java.util.Base64`
```java
Base64.getEncoder().encodeXxxxx()
Base64.getDecoder().decode()

Base64.getUrlEncoder()
Base64.getUrlDecoder()

Base64.getMimeEncoder()
Base64.getMimeDecoder()

```


### 并行数组
`parallelSort()`
parallexXxx系列的方法

## 新的Java工具
### Nashorn引擎：jjs
jjs是一个基于标准Nashorn引擎的命令行工具，可以接受js源码并执行。

### 类依赖分析器：jdeps
```sh
jdeps org.springframework.core-3.0.5.RELEASE.jar

org.springframework.core-3.0.5.RELEASE.jar -> C:\Program Files\Java\jdk1.8.0\jre\lib\rt.jar
   org.springframework.core (org.springframework.core-3.0.5.RELEASE.jar)
      -> java.io                                            
      -> java.lang                                          
      -> java.lang.annotation                               
      -> java.lang.ref                                      
      -> java.lang.reflect                                  
      -> java.util                                          
      -> java.util.concurrent                               
      -> org.apache.commons.logging                         not found
      -> org.springframework.asm                            not found
      -> org.springframework.asm.commons                    not found
   org.springframework.core.annotation (org.springframework.core-3.0.5.RELEASE.jar)
      -> java.lang                                          
      -> java.lang.annotation                               
      -> java.lang.reflect                                  
      -> java.util


```


## JVM的新特性
使用Metaspace（JEP 122）代替持久代（PermGen space）。
在JVM参数方面，使用 -XX:MetaSpaceSize和-XX:MaxMetaspaceSize代替原来的-XX:PermSize和-XX:MaxPermSize


## 附录
[Java SE Development Kit 8 Documentation](http://www.oracle.com/technetwork/java/javase/documentation/jdk8-doc-downloads-2133158.html)
[Java 8 教程汇总](https://wizardforcel.gitbooks.io/java8-tutorials/content/index.html)
title: Spring MVC（一） 常用点
date: 2015-08-06 17:37:37
tags: Spring MVC
categories: spring
---
springmvc一

## @RequestMapping URI 

### URI Template Patterns
[spring doc](http://docs.spring.io/spring/docs/4.2.1.BUILD-SNAPSHOT/spring-framework-reference/htmlsingle/#mvc-ann-requestmapping-uri-templates)

如何匹配这个url ` "/spring-web/spring-web-3.0.5.jar" ` ？

` @RequestMapping ` 支持正则表达式， 语法就是 ` {varName:regex} `。

比如上面那个url就可以这样
```java
@RequestMapping("/spring-web/{symbolicName:[a-z-]+}-{version:\\d\\.\\d\\.\\d}{extension:\\.[a-z]+}")
    public void handle(@PathVariable String version, @PathVariable String extension) {
        // ...
    }
}
```



#### Path Pattern Comparison
多重匹配优先规则：
* 越少数量URI variables 和 wild cards 的表达式越优先；例如` /hotels/{hotel}/* ` 比 ` /hotels/{hotel}/** ` 优先
* 数量一致，越长的表达式越优先；例如` /foo/bar* ` 比 ` /foo/* ` 优先
* 数量和长度一致， 越少wild cards的表达式越优先；例如` /hotels/{hotel} ` 比 ` /hotels/* ` 优先
* 例外的 ` /api/{a}/{b}/{c ` 比 ` /** ` 优先



### Path Pattern Matching By Suffix
默认Spring MVC会自动增加后缀 ` ".*" ` 来匹配URI。比如 ` /person ` 会自动映射成 ` /person.* ` ， 可以自动匹配文件类型，如` /person.pdf `, ` /person.xml `，不能匹配 ` /person.com` 
` .com ` 不是文件类型。



### Matrix Variables
需要修改下默认[配置](http://docs.spring.io/spring/docs/4.2.1.BUILD-SNAPSHOT/spring-framework-reference/htmlsingle/#mvc-ann-matrix-variables) ，，，，就可以使用[ "Matrix URIs"](http://www.w3.org/DesignIssues/MatrixURIs.html)

```
// GET /pets/42;q=11;r=22

@RequestMapping(path = "/pets/{petId}", method = RequestMethod.GET)
public void findPet(@PathVariable String petId, @MatrixVariable int q) {

    // petId == 42
    // q == 11

}
```

```
// GET /owners/42;q=11/pets/21;q=22

@RequestMapping(path = "/owners/{ownerId}/pets/{petId}", method = RequestMethod.GET)
public void findPet(
        @MatrixVariable(name="q", pathVar="ownerId") int q1,
        @MatrixVariable(name="q", pathVar="petId") int q2) {

    // q1 == 11
    // q2 == 22

}
```

```
// GET /pets/42

@RequestMapping(path = "/pets/{petId}", method = RequestMethod.GET)
public void findPet(@MatrixVariable(required=false, defaultValue="1") int q) {

    // q == 1

}
```

```
// GET /owners/42;q=11;r=12/pets/21;q=22;s=23

@RequestMapping(path = "/owners/{ownerId}/pets/{petId}", method = RequestMethod.GET)
public void findPet(
        @MatrixVariable Map<String, String> matrixVars,
        @MatrixVariable(pathVar="petId"") Map<String, String> petMatrixVars) {

    // matrixVars: ["q" : [11,22], "r" : 12, "s" : 23]
    // petMatrixVars: ["q" : 11, "s" : 23]

}
```


### BindingResult and @ModelAttribute
BindingResult必须直接跟在@ModelAttribute后面
``` 
@RequestMapping(method = RequestMethod.POST)
public String processSubmit(@ModelAttribute("pet") Pet pet, BindingResult result, Model model) { ... }
```

### 各种参数绑定

#### @PathVariable 
* 绑定URI的参数，若方法参数名称和需要绑定的uri template中变量名称不一致，需要在@PathVariable("name")指定uri template中的名称。

#### @RequestHeader
* 可以把Request请求header部分的值绑定到方法的参数
```
@RequestMapping("/displayHeaderInfo.do")  
public void displayHeaderInfo(@RequestHeader("Accept-Encoding") String encoding,  
                              @RequestHeader("Keep-Alive") long keepAlive)  {  
  
  //...  
  
}
```
    
#### @CookieValue
* 把Request header中关于cookie的值绑定到方法的参数  
  

#### @RequestParam
* 常用来处理简单类型的绑定，通过Request.getParameter() 获取的String可直接转换为简单类型的情况（ String--> 简单类型的转换操作由ConversionService配置的转换器来完成）；因为使用request.getParameter()方式获取参数，所以可以处理get 方式中queryString的值，也可以处理post方式中 body data的值；
* 用来处理Content-Type: 为 application/x-www-form-urlencoded编码的内容，提交方式GET、POST；
* 该注解有两个属性： value、required； value用来指定要传入值的id名称，required用来指示参数是否必须绑定  



#### @RequestBody
* 该注解常用来处理Content-Type: 不是application/x-www-form-urlencoded编码的内容，例如application/json, application/xml等；
* 它是通过使用HandlerAdapter 配置的HttpMessageConverters来解析post data body，然后绑定到相应的bean上的。
* 因为配置有FormHttpMessageConverter，所以也可以用来处理 application/x-www-form-urlencoded的内容，处理完的结果放在一个MultiValueMap<String, String>里，这种情况在某些特殊需求下使用，详情查看FormHttpMessageConverter api;  



#### @SessionAttributes
* 绑定HttpSession中的attribute对象的值

  
  
#### @ModelAttribute
该注解有两个用法，一个是用于方法上，一个是用于参数上；
用于方法上时：  通常用来在处理@RequestMapping之前，为请求绑定需要从后台查询的model；
用于参数上时： 用来通过名称对应，把相应名称的值绑定到注解的参数bean上；要绑定的值来源于：
* @SessionAttributes 启用的attribute 对象上；
* @ModelAttribute 用于方法上时指定的model对象；
* 上述两种情况都没有时，new一个需要绑定的bean对象，然后把request中按名称对应的方式把值绑定到bean中。


### HttpPutFormContentFilter
支持PUT方法


### @InitBinder
在controller里面配置web数据绑定
```
@Controller
public class MyFormController {

    @InitBinder
    public void initBinder(WebDataBinder binder) {
        SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd");
        dateFormat.setLenient(false);
        binder.registerCustomEditor(Date.class, new CustomDateEditor(dateFormat, false));
    }

    // ...
}
```

最新的Spring 4.2 里面
```
@Controller
public class MyFormController {

    @InitBinder
    public void initBinder(WebDataBinder binder) {
        binder.addCustomFormatter(new DateFormatter("yyyy-MM-dd"));
    }

    // ...
}
```


### @ControllerAdvice
被注解的类可以搭配` @ExceptionHandler`, ` @InitBinder`, and ` @ModelAttribute`，全局处理



## Resolving views
[Spring Doc : Resolving views](http://docs.spring.io/spring/docs/4.2.1.BUILD-SNAPSHOT/spring-framework-reference/htmlsingle/#mvc-viewresolver-resolver)



## [Spring’s multipart (file upload) support](http://docs.spring.io/spring/docs/4.2.1.BUILD-SNAPSHOT/spring-framework-reference/htmlsingle/#mvc-multipart)



## Handling exceptions
* HandlerExceptionResolver
* @ExceptionHandler

[Standard Spring MVC Exceptions](http://docs.spring.io/spring/docs/4.2.1.BUILD-SNAPSHOT/spring-framework-reference/htmlsingle/#mvc-ann-rest-spring-mvc-exceptions)
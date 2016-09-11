title: spring CORS
date: 2015-08-19 16:51:45
tags: Spring CORS
categories: spring
---

## CORS
跨域资源共享(CORS): 访问不同域站点上的资源；比如a.com上的某页面可以访问b.com上的资源，c.com上的某页面可以访问a.com上资源。

不转载了，链接： [CORS（跨域资源共享）简介 ](http://blog.csdn.net/hfahe/article/details/7730944)

## Spring framework 对 CORS 的支持
### controller上对类和方法的配置
Spring通过`@CrossOrigin`注解来实现对CORS的功能支持。

1. 方法级
```
@RestController
@RequestMapping("/account")
public class AccountController {

	@CrossOrigin
	@RequestMapping("/{id}")
	public Account retrieve(@PathVariable Long id) {
		// ...
	}

	@RequestMapping(method = RequestMethod.DELETE, path = "/{id}")
	public void remove(@PathVariable Long id) {
		// ...
	}
}
```

2. 类级
```
@CrossOrigin(origins = "http://domain2.com", maxAge = 3600)
@RestController
@RequestMapping("/account")
public class AccountController {

	@RequestMapping("/{id}")
	public Account retrieve(@PathVariable Long id) {
		// ...
	}

	@RequestMapping(method = RequestMethod.DELETE, path = "/{id}")
	public void remove(@PathVariable Long id) {
		// ...
	}
}
```

3. 混合
Spring结合类和方法的注解内容生成。
```
@CrossOrigin(maxAge = 3600)
@RestController
@RequestMapping("/account")
public class AccountController {

	@CrossOrigin("http://domain2.com")
	@RequestMapping("/{id}")
	public Account retrieve(@PathVariable Long id) {
		// ...
	}

	@RequestMapping(method = RequestMethod.DELETE, path = "/{id}")
	public void remove(@PathVariable Long id) {
		// ...
	}
}
```





### 全局配置
默认是GET，HEAD和POST有效
* java 配置


1. ` /**`路径， 全部默认属性
```
@Configuration
@EnableWebMvc
public class WebConfig extends WebMvcConfigurerAdapter {

	@Override
	public void addCorsMappings(CorsRegistry registry) {
		registry.addMapping("/**");
	}
}
```

2. 自定义路径和属性
```java
@Configuration
@EnableWebMvc
public class WebConfig extends WebMvcConfigurerAdapter {

	@Override
	public void addCorsMappings(CorsRegistry registry) {
		registry.addMapping("/api/**")
			.allowedOrigins("http://domain2.com")
			.allowedMethods("PUT", "DELETE")
			.allowedHeaders("header1", "header2", "header3")
			.exposedHeaders("header1", "header2")
			.allowCredentials(false).maxAge(3600);
	}
}
```



* XML 配置

1. ` /**`路径， 全部默认属性
```
<mvc:cors>
	<mvc:mapping path="/**" />
</mvc:cors>
```

2. 自定义属性
```
<mvc:cors>

	<mvc:mapping path="/api/**"
		allowed-origins="http://domain1.com, http://domain2.com"
		allowed-methods="GET, PUT"
		allowed-headers="header1, header2, header3"
		exposed-headers="header1, header2" allow-credentials="false"
		max-age="123" />

	<mvc:mapping path="/resources/**"
		allowed-origins="http://domain1.com" />

</mvc:cors>
```
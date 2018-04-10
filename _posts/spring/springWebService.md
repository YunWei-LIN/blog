title: spring WebService JAX-WS
date: 2015-08-25 19:51:43
tags: [spring,  WebService, JAX-WS]
categories: spring
---

spring提供完整的对标准 Java web services 的支持。有2种方式 SpringBeanAutowiringSupport 和 SimpleJaxWsServiceExporter 。

SpringBeanAutowiringSupport没实验成功 ：（ ， 下面只说 SimpleJaxWsServiceExporter 的方式。我是直接混合springMVC使用的。

* EndPoint
``` 
@Component("UserService") // auto scan
@WebService(serviceName="UserService")
public class UrUserEndpoint
{

    @Autowired
    IUrUserService        userService;
    
    @WebMethod
    public UrAbsUserEntity getUser(String username){
        
      return userService.findByUsername(username);
    }
    
}
```

* xml config
```
<bean class="org.springframework.remoting.jaxws.SimpleJaxWsServiceExporter">
    <property name="baseAddress" value="http://localhost:8081/"/> //注意，如果是直接混合springMVC使用的，不能和servlet容器端口冲突
</bean>
```


* java config
```
@Configuration
public class ApplicationConfiguration 
{
    @Bean
    public SimpleJaxWsServiceExporter jaxms()
    {
        SimpleJaxWsServiceExporter jaxws = new SimpleJaxWsServiceExporter();
        jaxws.setBaseAddress("http://localhost:8081/"); //注意，如果是直接混合springMVC使用的，不能和servlet容器端口冲突
        //此处可以有更多设置
        
        
        return jaxws;
    }

}
```

* wsdl
启动容器，在浏览器中访问 http://localhost:8081/UserService?wsdl , 就能看到wsdl的内容了

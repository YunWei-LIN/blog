---
title: Nginx + Tomcat + HTTPS
date: 2017-02-22 11:12:14
tags: 
 - nginx
 - tomcat
 - https
categories: server
---
主要内容
* 环境
* 证书
* nginx
* tomcat



......


本文基本参照 [ oschina 红薯 大大的文章 ](https://www.oschina.net/question/12_213459) 。
基本思路是 Nginx 上启用了 HTTPS，而 Nginx 和 Tomcat 之间走的却是普通的 HTTP 连接。

![](http://amadis.qiniudn.com/nginx_tomcat_ssl.png)

<!-- more -->

## 环境

主要以Centos为例。
```sh
# cat /etc/redhat-release
CentOS release 5.4 (Final)
```

## 证书
阿里云提供免费的`Symantec DV 证书`， 生成好后下载nginx的证书，包含 `xxxxxxx.pem` 和 `xxxxxxxxx.key`

## nginx配置
### 证书上传
将 `xxxxxxx.pem` 和 `xxxxxxxxx.key` 上传到 ngixn 的配置根目录 `/etc/nginx`。

### server

```nginxconf
server
{
        listen       443 ssl;
        listen       80 ;
        server_name  ssl.test.com  ;                                                                                                                                                                    
                                                                                                                                                                                                                  
        proxy_intercept_errors on;                                                                                                                                                                                
        charset utf-8;                                                                                                                                                                                            
                                                                                                                                                                                                                  
        # ssl                                                                                                                                                                                                     
        ssl_certificate   xxxxxxxxx.pem;                                                                                                                                                                    
        ssl_certificate_key  xxxxxxxxx.key;                                                                                                                                                                 
        ssl_session_timeout 5m;                                                                                                                                                                                   
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        ssl_ciphers AESGCM:ALL:!DH:!EXPORT:!RC4:+HIGH:!MEDIUM:!LOW:!aNULL:!eNULL;
        ssl_prefer_server_ciphers on;

        location /
        {
                proxy_pass http://localhost:8080;
                proxy_redirect off;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header Host $http_host;
                proxy_set_header X-Forwarded-Proto https;
        
        }

        location /status
        {
                stub_status on;
                access_log   off;
        }


        #  让http请求重定向到https请求   
        error_page 497  https://$host$uri?$args;  

        error_page  404              /404.html;
        location = /404.html {
          root /usr/share/nginx/html;
          internal;
        }
}
```

`497` 是nginx 内置 http 状态。
可参考[nginx强制使用https访问](http://blog.csdn.net/vfush/article/details/51086274)

## tomcat配置
主要配置 `server.xml`

```xml
<?xml version='1.0' encoding='utf-8'?>
<Server port="8005" shutdown="SHUTDOWN">
  <Service name="Catalina">
    <Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="443"
               proxyPort="443"/>

    <Engine name="Catalina" defaultHost="localhost">

      <Host name="localhost"  appBase="webapps"
            unpackWARs="true" autoDeploy="true">
            <Valve className="org.apache.catalina.valves.RemoteIpValve"
                  remoteIpHeader="x-forwarded-for"
                  remoteIpProxiesHeader="x-forwarded-by"
                  protocolHeader="x-forwarded-proto"
            />
            <Context path="" docBase="" reloadable="false"/>
      </Host>
    </Engine>
  </Service>
</Server>
```
特别特别注意的是必须有 <span style="color:red"> proxyPort="443"  redirectPort="443" </span>。
同时 `<Value>` 节点的配置也非常重要，否则你在 Tomcat 中的应用在读取 getScheme() 方法以及在 web.xml 中配置的一些安全策略会不起作用。

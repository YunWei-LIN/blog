title: nginx
date: 2016-01-22 10:48:42
tags: nginx
categories: server
---
主要内容

nginx


## 异常页面配置
* 异常页面存放
默认存放在 `/usr/share/nginx/html` 目录下。

* nginx配置文件

配置完类似下面的内容

```
server {
        listen 80 default_server;
        listen [::]:80 default_server ipv6only=on;

        . . .

        error_page 404 /40x.html;
        location = /40x.html {
                root /usr/share/nginx/html;
                internal;
        }
}
```

`注意` ： 如果配置多个 server， 需要在对应的server中配置 error_page 。

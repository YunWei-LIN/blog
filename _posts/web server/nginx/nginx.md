title: nginx
date: 2016-01-22 10:48:42
tags: nginx
categories: server
---
主要内容

nginx

<!-- more -->

## 安装
### yum源
如果系统自带yum源中没有nginx，按如下增加
```
# vim /etc/yum.repos.d/nginx.repo
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=0
enabled=1
```

### 安装
		# install
		yum -y install nginx
		
		# check
		nginx -v

## 配置

### 主配置文件
主配置文件`/etc/nginx/nginx.conf`, 增加如下内容
```
worker_rlimit_nofile 4096; # set open fd limit, 根据服务器具体情况


http {
 server_tokens  off; #隐藏nginx版本号

}
```

### 子配置
子配置文件目录`/etc/nginx/conf.d`, 文件名`*.conf`, `nj.mg.quanhouse.cn.conf` 为例
```
server
{
        listen       80;
        server_name nj.mg.quanhouse.cn;  #和tomcat的host name 或 alias name 一致。

        index index.html;
#       root  ;
        charset utf-8;

        location /
        {
                proxy_pass http://localhost:8080; # tomcat 端口
                proxy_redirect off;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header Host $http_host;
        }

        location /status
        {
                stub_status on;
                access_log   off;
        }

        

        #access_log off;
        #access_log  /data/logs/quanhouse.log  access;

}
```


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

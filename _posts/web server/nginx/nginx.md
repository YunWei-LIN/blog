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
+ yum

		# install
		yum -y install nginx
		
		# check
		nginx -v



+ 源码
确保 `Development Tools` 已经安装
```bash
wget http://nginx.org/download/nginx-1.11.2.tar.gz
    tar -xzvf nginx-1.11.2.tar.gz -C /usr/src
    # 安装依赖
    yum install -y zlib-devel.x86_64 pcre-devel.x86_64
    
    cd /usr/src/nginx-1.11.2
    # 以下是一行。。用于生成makefile。如果需要添加第三方模块，使用--add-module=/path/module1的方法编译
    ./configure --prefix=/usr/local/nginx 
    # make是生成在objs目录中，make install则安装到prefix所示的目录中
    make && make install
    # 没有错误出现的话，就可以进入nginx安装目录(/usr/local/nginx)配置。

```

成功安装后，进入 `/usr/local/nginx`
- conf：放置nginx相关的配置文件，最核心的是nginx.conf
- html：默认的网站根目录
- logs：日志文件目录(访问日志，错误日志，运行时的进程id cat logs/nginx.pid)
- sbin：主程序(nginx)目录

常用命令

        1、启动： /usr/local/nginx/sbin/nginx
        2、关闭： /usr/local/nginx/sbin/nginx -s stop
        3、重启： /usr/local/nginx/sbin/nginx -s reload
        4、指定(另外的)配置文件并启动(如果已经启动则报端口占用的错误)： /usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/new.conf
        5、查看pid： cat /usr/local/nginx/logs/nginx.pid，可以用于kill等操作
        6、查看安装时候的参数： /usr/local/nginx/sbin/nginx -V


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

## 负载均衡
在子配置文件中 

```bash
upstream xxxxServer {
        ip_hash;
            server 127.0.0.1:8080 ;
           # server 127.0.0.1:8000 weight=3;;
           # server 127.0.0.1:8080 down;

            server 127.0.0.2:8080 fail_timeout=600s  ;
           # server 127.0.0.2:8080 fail_timeout=600s  down;
    }

server
{
        listen       80;
        server_name  center.xxx.com ;
        #if ($host = '*.xxxx.com') {
        #        rewrite ^/(.*)$ http://m.xxxx.com/$1 permanent;
        #}


       # if ($http_referer ~* "bb01.win") {
       #         rewrite ^/(.*)$ http://m.xxxx.com/$1 permanent;
       # }


        charset utf-8;

        location /
        {
                proxy_pass http://xxxxServer;
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


       client_max_body_size 50M;

        error_page  404              /404.html;
        location = /404.html {
          root /usr/share/nginx/html;
          internal;
        }

        #access_log off;
        #access_log  /data/logs/cloudxn.log  access;



        location ~ .mp4 {
                    mp4;
                root /data/xxxx;
        }


        location ~ .apk {
                root /data/xxxx;
        }
}
     

```

nginx 的 upstream目前支持 4 种方式的分配 
+ 轮询（默认） 
      每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器down掉，能自动剔除。 
+ weight 
      指定轮询几率，weight和访问比率成正比，用于后端服务器性能不均的情况。 
+ ip_hash 
      每个请求按访问ip的hash结果分配，这样每个访客固定访问一个后端服务器，可以解决session的问题。  
+ least_conn
      最少连接。当某些请求需要更长时间来完成时，最少连接可以更公平的控制应用实例上的负载。


upstream 每个设备的状态:

+ down 表示单前的server暂时不参与负载 
+ weight  默认为1.weight越大，负载的权重就越大。 
+ max_fails ：允许请求失败的次数默认为1.当超过最大次数时，返回proxy_next_upstream 模块定义的错误 
+ fail_timeout:max_fails 次失败后，暂停的时间。 
+ backup： 其它所有的非backup机器down或者忙的时候，请求backup机器。所以这台机器压力会最轻。

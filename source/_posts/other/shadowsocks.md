title: shadowsocks
date: 2016-05-05 11:44:58
tags: shadowsocks
categories: 杂谈
---

关键词不说了， 知道ss的人就好，哈哈

<!-- more -->


## shadowsocks 安装
python版本安装

```
pip install shadowsocks
```

## server 配置
找到你自己的供应商，将server的信息 配置成 文件， 比如`gui-config.json`

## 启动
写了几个脚本，内容有点像这样的

* shadowsocks.sh
```
#!/bin/bash
cmd=$1

# start
if [ $cmd == "start" ]
then
  echo 'password' | sudo -S sslocal -c /xxxxxx/shadowsocks/gui-config.json -d start 
  #xxxxxx/shadowsocks 是 gui-config.json 的存放位置， gui-config.json 是上步的server配置信息
  #password 是你自己系统管理员密码，记得修改哦
  
#stop
elif [ $cmd == "stop" ]
then
  echo 'password' | sudo -S sslocal -c /xxxxxx/soft/shadowsocks/gui-config.json -d stop

# error
else
  echo "cmd only 'start' or 'stop'"
fi
```

* 启动命令
`./shadowsocks.sh start`

## http 全局代理
shadowsocks是socks5服务，在浏览器里使用一些插件可以满足浏览器使用了。一些其他场合，Privoxy实现Socks5转换为Http 服务。

### Privoxy

官方网站：http://www.privoxy.org/ 

* 下载 privoxy
比如目前最新的  privoxy-3.0.24-stable-src.tar.gz

* 解压

    tar xf privoxy-3.0.24-stable-src.tar.gz
    
* 编译安装


      cd privoxy-3.0.24-stable
	
      useradd privoxy
	
      autoheader && autoconf
      ./configure --prefix=你自己喜欢的安装路径， 装完后上面的源码文件夹可以删除
	
      make && make install


* 配置 
在 `--prefix=` 指定的路径中配置 `config` 文件；如果没指定， `config` 文件大概在 `/usr/local/etc/privoxy/`
  
  privoxy监听端口 
      listen-address 127.0.0.1:8118  
  查找`listen-address`， 端口可以修改
    
  shadowsocks 端口，（查找 关键字 forward-socks5t） 默认是这样的
      # forward-socks5t / 127.0.0.1:9050 .
	
      修改成 ==> 
	
      forward-socks5t / 127.0.0.1:1080 .

* 启动
编辑完成， copy 一份`config` 到 sbin目录下, 切换到sbin目录， 启动

      ./privoxy

### 环境变量
```
vim /etc/profile

##文件末尾加入：
 # add follows to the end (set proxy settings to the environment variables)
MY_PROXY_URL="http://127.0.0.1:8118/"
HTTP_PROXY=$MY_PROXY_URL
HTTPS_PROXY=$MY_PROXY_URL
FTP_PROXY=$MY_PROXY_URL
http_proxy=$MY_PROXY_URL
https_proxy=$MY_PROXY_URL
ftp_proxy=$MY_PROXY_URL
export HTTP_PROXY HTTPS_PROXY FTP_PROXY http_proxy https_proxy ftp_proxy
```

然后 `source` 下。

### 测试
访问下 gg，哈哈

      curl google.com
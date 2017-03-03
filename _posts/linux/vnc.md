
title: centos vnc远程桌面
date: 2016-10-10 19:29:44
tags: [linux, vnc]
categories: linux
---
主要内容
大部分情况下Linux的系统管理员都是通过SSH完成管理任务，特殊情况下可能会需要图形会话。VNC服务器就是这样一个自由开源软件，他可以让用户远程访问服务器的桌面环境。
下面就是关于CentOS 7 上配置VNC服务的内容。

<!-- more -->

## 环境
```
cat /etc/redhat-release #only redhat release
CentOS Linux release 7.2.1511 (Core)
```

## 安装
```
yum -y install tigervnc-server tigervnc
```

## 配置
### 配置文件
* 创建一个新的配置文件
 ``` 	
  cp /lib/systemd/system/vncserver@.service /lib/systemd/system/vncserver@:1.service
```

* 编辑`vncserver@:1.service`
根据远程桌面用户修改配置文件， 替换`<USER>`部分;
Type改成`simple`，默认的`forking`可能导致服务启动失败。

	```
	[Unit]
	Description=Remote desktop service (VNC)
	After=syslog.target network.target

	[Service]
	## Type=forking
	Type=simple

	# Clean any existing files in /tmp/.X11-unix environment
	ExecStartPre=/bin/sh -c '/usr/bin/vncserver -kill %i > /dev/null 2>&1 || :'
	## ExecStart=/usr/sbin/runuser -l <USER> -c "/usr/bin/vncserver %i"
	## PIDFile=/home/<USER>/.vnc/%H%i.pid
	ExecStart=/usr/sbin/runuser -l root -c "/usr/bin/vncserver %i"
	PIDFile=/root/.vnc/%H%i.pid
	ExecStop=/bin/sh -c '/usr/bin/vncserver -kill %i > /dev/null 2>&1 || :'

	[Install]
	WantedBy=multi-user.target
	```

### 更新systemctl
```
systemctl daemon-reload
```

### VNC密码
设置对应远程桌面用户的VNC密码， 此密码不同于用户登录密码。
```
vncserver
```

如果需要更多用户连接，那么需要再创建配置文件和端口， 就是重复上面的步骤；如创建`vncserver@:2.service`， 并替换里面的用户名。

## 服务
### 启动
```
systemctl start vncserver@:1.service
```


### 开机自启动
```
systemctl enable vncserver@:1.service
```


## 其他
### 其他命令
```
vncserver -list #显示
vncserver :n #启动
vncserver -kill n #关闭
```

### 客户端
VNC本身服务默认使用5900， 不同用户使用VNC会获得不同的端口（都是5900的子端口），配置文件名里的数字就指定改用户使用5900的具体子端口，如上面root用户就使用5901端口。

客户端连接时需要采用`ip:端口`格式。

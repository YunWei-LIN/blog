title: docker 入门 （未完）
date: 2017-10-10 19:44:58
tags: [docker]
categories: docker
---

docker 入门 （未完）

*更新历史*
无
<!-- more -->

---------------------------------------------------------------------

# 镜像
## Dockerfile
不要使用 dockercommit 定制镜像,定制行为应该使用 Dockerfile 来完成。
Dockerfile中每一个指令都会建立一层

+ run
使用 `&&` 将各个所需命令串联起来

+ 镜像构建上下文(Context)
`docker build [选项] <上下文路径/URL/->`
` docker build -t nginx:v3 .`
最后有一个`.`。 `.`表示当前目录, 是在指定上下文路径。 即 docker 所能操作的根目录， 超出其路径， docker 是不能操作的
`docker build`  命令得知这个路径后,会将路径下的所有内容打包,然后上传给 Docker 引擎。这样 Docker 引擎收到这个上下文包后,展开就会获得构建镜像所需的一切文件。
初学的错误是将`Dockerfile`放到硬盘根目录, 把上下文设置成硬盘根目录， 结果发现`docker build`执行后,在发送一个几十GB的东西。
一般来说,应该会将`Dockerfile`置于一个空目录下,或者项目根目录下。如果该目录下没有所需文件,那么应该把所需文件复制一份过来。
如果目录下有些东西确实不希望构建时传给 Docker 引擎,那么可以用`.gitignore`一样的语法写一个`.dockerignore`,该文件是用于剔除不需要作为上下文传递给Docker 引擎的。
那么为什么会有人误以为`.`是指定`Dockerfile`所在目录呢?这是因为在默认情况下,如果不额外指定`Dockerfile`的话,会将上下文目录下的名为 Dockerfile  的文件作为 Dockerfile。

### COPY复制文件
### ADD更高级的复制文件
### CMD容器启动命令
### ENTRYPOINT入口点
### ENV设置环境变量
### ARG构建参数
### VOLUME定义匿名卷
### EXPOSE声明端口
### WORKDIR指定工作目录
### WORKDIR指定工作目录
### USER指定当前用户
### HEALTHCHECK健康检查
### ONBUILD为他人做嫁衣裳


## 删除本地镜像
`dockerrmi[选项]<镜像1>[<镜像2>...]`
其中, <镜像>可以是  镜像短ID、  镜像长ID、  镜像名或者  镜像摘要。
可以使用 `dockerimages-q`来配合使用`dockerrmi`,这样可以成批的删除希望删除的镜像。


# 容器
## 新建并启动
`docker	run`
`	sudo	docker	run	-t	-i	ubuntu:14.04	/bin/bash`

检查本地是否存在指定的镜像,不存在就从公有仓库下载启动
利用镜像创建并启动一个容器
分配一个文件系统,并在只读的镜像层外面挂载一层可读写层
从宿主主机配置的网桥接口中桥接一个虚拟接口到容器中去
从地址池配置一个	ip	地址给容器
执行用户指定的应用程序
执行完毕后容器被终止


## 启动已终止容器
`docker	start`

## 后台(background)运行
`sudo	docker	run	-d	ubuntu:14.04	/bin/sh	-c	"while	true;	do	echo	hello	world;	sleep	1;	done`
容器会在后台运行并不会把输出的结果(STDOUT)打印到宿主机上面(输出结果可以用docker	logs	查看)。
`sudo	docker	logs	[container	ID	or	NAMES]`

## 终止容器
`	docker	stop	 `

## 进入容器
+ `nsenter`	命令

```sh
PID=$(docker	inspect	--format	"{{	.State.Pid	}}"	<container>)
nsenter	--target	$PID	--mount	--uts	--ipc	--net	--pid
nsenter	--target	$pid	--mount	--uts	--ipc	--net	--pid		--	/usr/bin/env	\	--ignore-environment	HOME=/root	/bin/bash	--login
```

+ `.bashrc_docker`
```sh
wget	-P	~	https://github.com/yeasy/docker_practice/raw/master/_local/.bashrc_docker;
echo	"[	-f	~/.bashrc_docker	]	&&	.	~/.bashrc_docker"	>>	~/.bashrc;	source	~/.bashrc
```

这个文件中定义了很多方便使用	Docker	的命令,例如	 	docker-pid		可以获取某
个容器的	PID;而	 	docker-enter		可以进入容器或直接在容器内执行命令。

```sh
echo	$(docker-pid	<container>)
docker-enter	<container>	ls
```

## 导出和导入容器 `docker	export`
### 导出 `docker	export`

### 导入 `docker	import`
```
cat	ubuntu.tar	|	sudo	docker	import	-	test/ubuntu:v1.0
sudo	docker	import	http://example.com/exampleimage.tgz	example/imagerepo
```

### 删除容器
`sudo	docker	rm		trusting_newton`
如果要删除一个运行中的容器,可以添加	 	-f		参数

### 清理所有处于终止状态的容器
`docker	rm	$(docker	ps	-a	-q)`可以全部清理掉, 包括还在运行中的容器.


# Docker	数据管理
## 数据卷




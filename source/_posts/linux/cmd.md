title: linux 常用规则， 酷～～～
date: 2015-07-20 10:09:38
tags: linux
categories: linux
---
## 命令
* 查看整体空间 (-h 按M单位)
```
$ df -h
```


* 查看输出当前目录下各个子目录所使用的空间(-h 按M单位)
``` bash
$ du -h --max-depth=1
```

排序
```
$ du  --max-depth=1 |sort -rn
```

* 手动清空缓存
``` bash
$ sync
$ echo 3 > /proc/sys/vm/drop_caches
```

* 查找以前用过的命令
`ctrl + r`


* Netstat
Netstat 命令用于显示各种网络相关信息，如网络连接，路由表，接口状态 (Interface Statistics)，masquerade 连接，多播成员 (Multicast Memberships) 等等。
[Linux netstat命令详解 链接](http://www.cnblogs.com/ggjucheng/archive/2012/01/08/2316661.html)
  程序端口占用情况
```
$ netstat –apn 

$ netstat   -anp   |   grep  portno

$ ps -aux | grep java
```

* 抓包 tcpdump / wireshark

* rsync 增量备份，保持原有属性
-a
-z
-p
--delete

* inotify + rsync 实时备份
inotify 安装

inotifywait
-e 监控事件
-m 持续监控
-r 递归
-q 简化信息



* ctrl + r 查找历史命令

* rs 上传

* waf ： web application firewall
modsecurity

* Linux下高效数据恢复软件extundelete

* curl wget lynx
[curl](http://blog.51yip.com/linux/1049.html)
```
$ curl -I ip/domain #查看web服务器类型和版本
$ wget -C URL #断点续传 -t 重试次数 -T 超时时间
$ wget --user username --password pass URL #认证
$ curl -C URL #断点续传
```

## 常用技巧
### 特殊变量
|变量|含义|
|------|---------------------------------------------|
|$0|当前脚本的文件名|
|$n|传递给脚本或函数的参数。n 是一个数字，表示第几个参数。例如，第一个参数是$1，第二个参数是$2。|
|!$|上个命令最后一个参数|
|$#|传递给脚本或函数的参数个数。|
|$*|传递给脚本或函数的所有参数。|
|$@|传递给脚本或函数的所有参数。被双引号(" ")包含时，与 $* 稍有不同，下面将会讲到。|
|$?|上个命令的退出状态，或函数的返回值。|
|$$|当前Shell进程ID。对于 Shell 脚本，就是这些脚本所在的进程ID。|
|$!|执行上一个后台程序的PID|
`$*` 和 `$@` 都表示传递给函数或脚本的所有参数，不被双引号(" ")包含时，都以"$1" "$2" … "$n" 的形式输出所有参数。
但是当它们被双引号(" ")包含时，`$*` 会将所有的参数作为一个整体，以"$1 $2 … $n"的形式输出所有参数；`$@` 会将各个参数分开，以"$1" "$2" … "$n" 的形式输出所有参数。

###   常用PATTERN： 
    `^#`:   以#开头
    `#$`:   以#结尾
    `^$`:   空行

## [知乎](https://www.zhihu.com/question/20140085)

## 基础知识
### 最小存储单位
|名称 |     最小存储单位|
|---------|-------------|
|硬盘      |  扇区（512B）|
|文件系统   |  block（1K，4K）|
|RAID     |  chunk （512）    mdadm  -c|
|LVM      |  PE （16M 自定义）|




## 调教 kaffeine 播放各类媒体格式
kaffeine 是基于 `libxine2`, openSUSE 官方提供的 libxine2 缺少一些插件，那用户要怎么安装这些编解码器呢？ 我们先说安装 mkv, mp4, wmv, m4v, mov, 3gp, 3g2 等视频文件的编解码器，这些可以很方便的从 packman 安装。核心的包是 `libxine2-codecs` ，我以 openSUSE 13.2 为例进行讲解：
```bash
sudo zypper ar -f http://packman.inode.at/suse/openSUSE_13.2/ packman
sudo zypper update
sudo zypper install libxine2-codecs
```


## 安全相关
`last`
`lastb`
`lastlog`
`history`
`/var/log/secure` : 登录的ssh公钥签名可以通过 `ssh-keygen -l -f xxx.pub` 获得 
`远程日志服务`

title: 1-3 rhel7基本命令和操作
date: 2015-11-27 15:29:44
tags: GOD linux
categories: linux
---
## 基础知识 
每个linux发行版有不一致，本文以rhel7.1 为准。
* 图形界面和字符界面相互切换
linux终端tyy1 ~ tyy6, tyy1 就是图形界面;
Ctrl + Alt + F1～6 （根据笔记本灵活试试）

* 虚拟终端 Terminal  pts
Ctrl + shift + T （在Terminal焦点中才起作用，可看看terminal的菜单）

<!--more-->

* 内部命令 外部命令
随系统启动就加载到内存就是内部命令，反之就是外部命令。可通过 `type` 查看。
```bash
$ type cd
cd is a shell builtin # 内部命令
$ type vim
vim is /usr/bin/vim # 外部命令
```

* linux系统中不同的颜色代表了不同的文件类型

|  颜色 |  类型 |  例子| 
|-------|-----------|------------------|
|  蓝色 | 目录 | /etc | 
|  黑色 | 文件 | /etc/passwd | 
|  浅蓝色 | 链接 | /etc/grub2.cfg | 
|  红色 | 压缩包 | boot.tar.gz | 
|  绿色 | 可执行文件 | /etc/init.d/network | 
|  黑底黄字 | 设备文件 | /dev/sda | 


* bash shell 提示符
root 为 `#` ， 一般用户为 `$`

## 常用命令
Linux命令输入规律：
空格作为分割

| 命令名      | [选项]（[参数]）       | [选项的值]（[参数的值]）| 
|------------|-----------|------------------| 
| rm | -rf | /etc/passwd | 

### hwclock
查看系统和BIOS硬件时间：  指的是bios时间  （格里尼兹天文台）
```
# hwclock 
2015年11月27日 星期五 17时53分39秒  -0.344261 秒
```

### 系统时间
* 查看
```
# date
2015年 11月 27日 星期五 18:05:22 CST
# date "+%F %R"
2015-11-27 18:05
```

* 修改
```
# date -s "2016-01-01 01:01"
```

### 获取帮助
* 加参数-h 或--help

* man

### 关机命令
shutdown、 init 、reboot   poweroff
* shutdown
作用：关机，重启，定时关机
语法：shutdown  [选项]
-r     => 重新启动计算机
-h    => 关机
-h  时间  =>定时关机
-c    => 取消之前的定时关机  或ctrl+c


* init 
作用：切换系统运行级别
语法：init  0-6

## 启动级别配置
Linux 7个启动级别：
0 系统停机模式，系统默认运行级别不能设置为0，否则不能正常启动，机器关闭。
1 单用户模式，root权限，用于系统维护，禁止远程登陆，就像Windows下的安全模式登录。
2 多用户模式，没有NFS网络支持。
3 完整的多用户文本模式，有NFS，登陆后进入控制台命令行模式。
4 系统未使用，保留一般不用，在一些特殊情况下可以用它来做一些事情。例如在笔记本电脑的电池用尽时，可以切换到这个模式来做一些设置。
5 图形化模式，登陆后进入图形GUI模式，X Window系统。
6 重启模式，默认运行级别不能设为6，否则不能正常启动。运行init 6机器就会重启


* RHEL7不再使用/etc/inittab文件进行默认的启动级别配置
systemd使用比sysvinit的运行级更为自由的target替代。第3运行级用multi-user.target替代。第5运行级用graphical.target替代。runlevel3.target和runlevel5.target分别是指向 multi-user.target和graphical.target的符号链接。
切换到第3运行级
```
# systemctl isolate multi-user.target
# systemctl isolate runlevel3.target
```
切换到第5运行级
```
# systemctl isolate graphical.target
# systemctl isolate runlevel5.target
```

* 设置默认的运行界别
systemd使用链接来指向默认的运行级别。
在创建新的链接前，可以通过下面命令删除存在的链接： 
```
rm /etc/systemd/system/default.target
  
#默认启动级别3
systemctl -f enable multi-user.target
systemctl set-default multi-user.target
  
#默认启动级别5
systemctl -f enable graphical.target
systemctl set-default graphical.target
```
可以参照 `/etc/inittab` 文件

* 查看运行级别
```
# runlevel
```

## 配置实验环境
### 配置IP地址
RHEL7中弱化了setup的功能，对于网络管理来说，主要通过nmtui修改网络配置（RHEL7默认安装，前提是需要开启NetworkManager.service 才可以使用）
```
# systemctl status NetworkManager
NetworkManager.service - Network Manager
   Loaded: loaded (/usr/lib/systemd/system/NetworkManager.service; enabled)
   Active: active (running) since Fri 2015-11-27 03:27:10 EST; 1h 52min ago
 Main PID: 1158 (NetworkManager)
   CGroup: /system.slice/NetworkManager.service
           ├─1158 /usr/sbin/NetworkManager --no-daemon
           └─1777 /sbin/dhclient -d -q -sf /usr/libexec/nm-dhcp-helper -pf /v...

Nov 27 03:27:11 localhost.localdomain NetworkManager[1158]: <info>  (eno16777...
Nov 27 03:27:11 localhost.localdomain NetworkManager[1158]: <info>  (eno16777...
Nov 27 03:27:11 localhost.localdomain NetworkManager[1158]: <info>  NetworkMa...
Nov 27 03:27:11 localhost.localdomain NetworkManager[1158]: <info>  NetworkMa...
Nov 27 03:27:11 localhost.localdomain NetworkManager[1158]: <info>  Policy se...
Nov 27 03:27:11 localhost.localdomain NetworkManager[1158]: <info>  (eno16777...
Nov 27 03:27:15 localhost.localdomain NetworkManager[1158]: <info>  startup c...
Nov 27 03:27:43 localhost.localdomain NetworkManager[1158]: <info>  (eno16777...
Nov 27 03:27:43 localhost.localdomain NetworkManager[1158]: <info>  (eno16777...
Nov 27 03:27:43 localhost.localdomain NetworkManager[1158]: <info>  (eno16777...
Hint: Some lines were ellipsized, use -l to show in full.
```

运行 ` nmtui `， 图形界面。

### 关闭防火墙
```
# systemctl stop firewalld.service
# systemctl disable firewalld.service 
rm '/etc/systemd/system/dbus-org.fedoraproject.FirewallD1.service'
rm '/etc/systemd/system/basic.target.wants/firewalld.service'
```
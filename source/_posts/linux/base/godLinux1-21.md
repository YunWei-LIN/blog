title: 1-21 RHEL7网络管理
date: 2015-12-23 16:29:44
tags: GOD linux
categories: linux
---
## 目录
OSI七层模型和TCP/IP四层模型
网络相关协议
TCP三次握手和四次挥手
网络管理相关命令
tcpdump和wireshark抓包

<!-- more -->

## 网络模型
![](http://7xklqw.com1.z0.glb.clouddn.com/OSI&TcpIp.png)

## 网络协议
TCP/IP协议
TCP/IP协议是一个协议簇。里面包括很多协议的

例如：
超文本传输协议(HTTP):万维网的基本协议. 
文件传输ftp (TFTP简单文件传输协议):   搭建无人值守安装服务器
远程登录(Telnet)
网络管理(SNMP简单网络管理协议)
TCP TransmissionControl Protocol，传输控制协议）是面向连接的协议，
UDP UDP（User Data Protocol，用户数据报协议）非连接的协议
Internet协议(IP)
Internet控制信息协议(ICMP) 
地址解析协议(ARP)
反向地址解析协议(RARP)

TCP和UDP区别
1.基于连接与无连接；
2.对系统资源的要求（TCP较多，UDP少）；
3.UDP程序结构较简单；
4.tcp流模式与udp数据报模式 ；


## TCP三次握手和四次挥手
![](http://7xklqw.com1.z0.glb.clouddn.com/3hands.png)
![](http://7xklqw.com1.z0.glb.clouddn.com/4hands.png)

## 网络管理相关命令
### mii-tool
* 作用：查看网卡物理连接是否正常
* 语法：mii-tool 网卡名
```bash
[root@localhost ~]#mii-tool eno18345712
eno18345712: negotiated1000baseT-FD flow-control, link ok
[root@localhost~]# mii-tool eno18345712
eno18345712:no link
```

### ethtool
* 作用：查询及设置网卡参数
* 语法：ethtool 网卡名


### mii-tool
* 作用：查看网卡物理连接是否正常
* 语法：mii-tool 网卡名
```bash
[root@localhost ~]#mii-tool eno18345712
eno18345712: negotiated1000baseT-FD flow-control, link ok
[root@localhost~]# mii-tool eno18345712
eno18345712:no link
```

### 配置网络和IP地址
#### 方法一：nmtui
图形界面
* 开启NetworkManager
此服务不开启，则无法通过nmtui工具配置网络,在RHEL7中增强了NetworkManager服务功能，弱化了network的功能，RHEL7中要确定NetworkManager服务是开启的状态
```bash
[root@localhost ~]# systemctl restart NetworkManager
[root@localhost ~]# systemctl enable NetworkManager
```

* 网卡的名称
`[root@localhost ~]#ifconfig -a`

* nmtui 图形操作
```
[root@localhost ~]#nmtui
```
  ![](http://7xklqw.com1.z0.glb.clouddn.com/nmtui1.png)
  ![](http://7xklqw.com1.z0.glb.clouddn.com/nmtui2.png)











#### 方法二：修改网卡配置文件
```bash
[root@localhost ~]# vim /etc/sysconfig/network-scripts/ifcfg-eno18345712 #ifcfg-eno18345712 看服务器的实际名称
TYPE="Ethernet"
BOOTPROTO="none"
DEFROUTE="yes"
IPV4_FAILURE_FATAL="no"
IPV6INIT="yes"
IPV6_AUTOCONF="yes"
IPV6_DEFROUTE="yes"
IPV6_FAILURE_FATAL="no"
NAME="eno18345712"
UUID="2f532aa0-e1cf-4fa4-8379-86c911727689"
DEVICE="eno18345712"
ONBOOT="yes"                                                     #开启网卡
IPADDR="192.168.1.68"
PREFIX="24"
GATEWAY="192.168.1.1"
DNS1="8.8.8.8"
IPV6_PEERDNS="yes"
IPV6_PEERROUTES="yes"
IPV6_PRIVACY="no"
```
注：
BOOTPROTO=static   静态IP
BOOTPROTO=dhcp   动态IP
BOOTPROTO=none   无（不指定）


### 修改主机名 hostname
* 作用：修改主机名
```
[root@localhost ~]# vim /etc/hostname
[root@localhost ~]# hostname
```

### 配置hosts
* 作用：配置IP与主机名（域名）的对应
```bash
[root@localhost ~]# vim /etc/hosts
    127.0.0.1   localhost localhost.localdomain localhost4localhost4.localdomain4
    ::1         localhost localhost.localdomainlocalhost6 localhost6.localdomain6
```

### 配置DNS
* 作用：配置DNS
```
[root@localhost ~]# vim /etc/resolv.conf
```


### 域名解析顺序
* 作用：配置域名解析顺序，优先解析hosts还是解析DNS
```bash
[root@localhost ~]# vim /etc/nsswitch.conf
  
查找以下内容
#hosts:     db files nisplus nis dns
hosts:      files dns #根据需要修改files dns的位置
```

### 配置服务的端口号
* 作用：可以编辑查看常用端口对应的名字。iptables或netstat要把端口解析成协议名时，都需要使用到这个文件。另外xinetd服务管理一些小服务时，也会使用到此文件来查询对应的小服务端口号。
```
[root@localhost ~]# vim /etc/services
```

### 路由信息 route
* 查看默认网关
  ```bash
  [root@localhost ~]# route -n
  Kernel IP routing table
  Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
  0.0.0.0         192.168.1.1     0.0.0.0         UG   100    0       0 eno16777736
  192.168.1.0     0.0.0.0         255.255.255.0   U    100    0        0 eno16777736
  192.168.2.0     0.0.0.0         255.255.255.0   U    100    0        0 eno33554984
  ```

  说明：
  route命令输出的路由表字段含义如下：
      Destination 目标
	    The destination networkor destination host. 目标网络或目标主机。

      Gateway 网关
	    The gateway address or'*' if none set. 网关地址，如果没有就显示星号或4个0。

      Genmask 网络掩码
	    The  netmask for  the  destination net; '255.255.255.255' for a
	    host destination and'0.0.0.0' for the default route.

* 添加/删除路由条目：
  增加 (add) 与删除 (del) 路由的相关参数：
    `-net`    ：表示后面接的路由为一个网域；
    `-host`  ：表示后面接的为连接到单部主机的路由；
    `netmask` ：与网域有关，可以设定 netmask 决定网域的大小；
    `dev`    ：如果只是要指定由那一块网路卡连线出去，则使用这个设定，后面接 eth0 等
  ```
  [root@localhost ~]# route add -net 192.168.2.0 netmask 255.255.255.0 dev eno33554984
    
  [root@localhost ~]# route del -net 192.168.2.0 netmask 255.255.255.0
  ```

### 网络连接状态 netstat
* 作用： 网络连接状态
* 用法： netstat [参数]
  参数：
  -a,--all
  -n,--numeric              don't resolvenames
  -p,--programs
  -t  显示tcp连接
  -u  显示udp连接
```
[root@localhost ~]# netstat -anptu
```

### 网络诊断 ping
* 作用： 帮助分析和判定网络故障
* 用法： ping [参数]  IP 
参数：
-c 数目 在发送指定数目的包后停止。
-i 秒数 设定间隔几秒送一个网络封包给一台机器，预设值是一秒送一次。
-I  指定接口，指定从哪个端口（网卡）出去。
```
[root@localhost ~]# ping -I eno16777736 192.168.1.1
```

### 网络流量 Iptraf
* 作用： 监控网络流量
```
[root@localhost ~]# rpm -ivh /mnt/Packages/iptraf-ng-1.1.4-4.el7.x86_64.rpm
[root@localhost ~]# iptraf-ng
```

### 检查IP地址 arping
* 作用：查看IP地址是否有冲突
```
[root@localhost ~]# arping -I eno16777736 192.168.1.1
ARPING 192.168.1.1 from192.168.1.68 eno16777736
Unicast reply from 192.168.1.1[FC:D7:33:24:88:24]  1.102ms
Unicast reply from 192.168.1.1[FC:D7:33:24:88:24]  0.801ms
Unicast reply from 192.168.1.1[FC:D7:33:24:88:24]  0.741ms
```



## 抓包工具
### tcpdump
* 作用： dump the traffic on a network，根据使用者的定义对网络上的数据包进行截获的包分析工具
* 用法： tcpdump 
  port  端口号
  -c  抓几个包
  -n  不解析端口号为协议名
  -S    Print absolute, rather than relative, TCP  sequence numbers.
  -i   指定网卡
```
[root@localhost ~]#tcpdump  port 22 -c  3  -n -S  -i  eno16777736
```

### wireshark
作用： 网络封包分析软件。网络封包分析软件的功能是撷取网络封包，并尽可能显示出最为详细的网络封包资料。
```bash
#安装
[root@localhost ~]# yum -y install wireshark
  
#创建一个保存抓包信息的文件
[root@localhost ~]# touch  a.txt
  
#执行抓包命令
[root@localhost ~]# tshark -w a.txt -i eno16777736

#分析
[root@localhost ~]# tshark -r a.txt
```
`-r` 指定要读取的包文件
`-V` 将包尽可能的解析（这个有时在包数量很多的情况下可以不使用，这样它会给出一个很简洁的报文解释）










---
title: CentOS 7 Firewalld
date: 2017-03-28 19:11:43
tags: [linux, fail2ban]
categories: linux
---

主要内容
CentOS 7 Firewalld 防火墙介紹

主要内容
* 基本概念
* 基础命令
* 具体规则

CentOS 7 之后改用 firewalld 以 zone 的区域分割来建立，并可动态设置方式执行避免中断的问题。

<!-- more -->

## 基本概念
### Firewalld 相关路径
`/etc/firewalld`：设定档路径
`/usr/bin/`：firewall-cmd 指令所在的位置
`/usr/lib/firewalld/`：firewalld 设定的设定资源 (xml 格式)，例如 nfs.xml 就定义了 <port protocol="tcp" port="2049"/>
`/usr/lib/firewalld/services/`： 所有服务xml

共有这些服务，可以直接使用

    amanda-client.xml high-availability.xml kpasswd.xml mysql.xml pop3s.xml smtp.xml
    bacula-client.xml https.xml ldaps.xml nfs.xml postgresql.xml ssh.xml
    bacula.xml http.xml ldap.xml ntp.xml proxy-dhcp.xml telnet.xml
    dhcpv6-client.xml imaps.xml libvirt-tls.xml openvpn.xml radius.xml tftp-client.xml
    dhcpv6.xml ipp-client.xml libvirt.xml pmcd.xml RH-Satellite-6.xml tftp.xml
    dhcp.xml ipp.xml mdns.xml pmproxy.xml rpc-bind.xml transmission-client.xml
    dns.xml ipsec.xml mountd.xml pmwebapis.xml samba-client.xml vnc-server.xml
    ftp.xml kerberos.xml ms-wbt.xml pmwebapi.xml samba.xml wbem-https.xm


### zone
Firewall 能将不同的网络连接归类到不同的信任级别，Zone 提供了以下几个级别

+ drop: 丢弃所有进入的包，而不给出任何响应
+ block: 拒绝所有外部发起的连接，允许内部发起的连接
+ public: 允许指定的进入连接
+ external: 同上，对伪装的进入连接，一般用于路由转发
+ dmz: 允许受限制的进入连接
+ work: 允许受信任的计算机被限制的进入连接，类似 workgroup
+ home: 同上，类似 homegroup
+ internal: 同上，范围针对所有互联网用户
+ trusted: 信任所有连接


最常用 
`public`： 公開的場所，不信任網域內所有連線，只有被允許的連線才能進入，一般只要设定這裡就可以了
`dmz`： (Demilitarized Zone) 非軍事區，允許對外連線，內部網路只有允許的才可以連線進來




所謂的 zone 就表示主机位于哪个环境区域，需要设定哪些规则，在 firewalld 共有 7 個zones
> 先决定主机要设定在哪个区域 zone >> 再往该 zone 设定规则 >>> 重新读取设定 sudo firewall-cmd --reload


### 过滤规则

+ source: 根据源地址过滤
+ interface: 根据网卡过滤
+ service: 根据服务名过滤
+ port: 根据端口过滤
+ icmp-block: icmp 报文过滤，按照 icmp 类型配置
+ masquerade: ip 地址伪装
+ forward-port: 端口转发
+ rule: 自定义规则

其中，过滤规则的优先级遵循如下顺序

+ source
+ interface
+ firewalld.conf


## 基础命令

```
systemctl start firewalld         # 启动
systemctl enable firewalld        # 开机启动
systemctl stop firewalld          # 关闭
systemctl disable firewalld       # 取消开机启动
```

## 具体规则/例子
### 显示目前的设定
    firewall-cmd --list-all
 
### 关掉 DHCP 服务 port

    firewall-cmd --zone=public --remove-service dhcpv6-client

### 暂时开启 DNS port 53

    systemctl start named
    systemctl enable named
    firewall-cmd --add-service=dns
    firewall-cmd --reload
    firewall-cmd --list-all

### 永久开启 DNS port 53
    firewall-cmd --add-service=dns --permanent
    firewall-cmd --reload

### 如何修改主机的设定 zone
    vi /etc/firewalld/firewalld.conf
    > 修改 DefaultZone=public
    > 变成 DefaultZone=dmz
    firewall-cmd --reload

### 加入自行指定的连接
    firewall-cmd --add-port=8080/tcp --permanent
    success
    firewall-cmd --reload
    success
    firewall-cmd --list-all

### 限制某服务只能从哪IP进入
    firewall-cmd --add-rich-rule="rule family="ipv4" source address="192.168.1.88" service name="ssh" accept"
    
或 ip subnet


    firewall-cmd --add-rich-rule="rule family="ipv4" source address="192.168.1.0/24" service name="ssh" accept"

### 限制某 port 只能从哪IP进入
    firewall-cmd --add-rich-rule="rule family="ipv4" source address="192.168.12.9" port port="8080" protocol="tcp" accept"
注意：port port="8080" 不是写错唷！這是他的格式

### 从 zone 移除某項服务
    firewall-cmd --zone=public --add-service=http --permanent
    firewall-cmd --zone=public --remove-service=http --permanent

    firewall-cmd --zone=public --add-port=8080/tcp --permanent
    firewall-cmd --zone=public --remove-port=8080/tcp --permanent

## 参考资料
[CentOS 7 Firewalld 防火牆說明介紹](http://blog.xuite.net/tolarku/blog/363801991-CentOS+7+Firewalld+%E9%98%B2%E7%81%AB%E7%89%86%E8%AA%AA%E6%98%8E%E4%BB%8B%E7%B4%B9)
[CentOS 7 下使用 Firewall](https://havee.me/linux/2015-01/using-firewalls-on-centos-7.html)
[fedora FirewallD](https://fedoraproject.org/wiki/FirewallD)

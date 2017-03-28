---
title: CentOS 7 Firewalld
date: 2017-03-28 18:11:43
tags: [linux, fail2ban]
categories: linux
---

主要内容
CentOS 7 Firewalld 防火墙介紹

CentOS 7 之后改用 firewalld 以 zone 的区域分割来建立，并可动态设置方式执行避免中断的问题。

<!-- more -->


Firewalld 相關路徑
/etc/firewalld：設定檔路徑
/usr/bin/：firewall-cmd 指令所在的位置
/usr/lib/firewalld/：firewalld 預設的設定資料 (xml 格式)，例如 nfs.xml 就定義了 <port protocol="tcp" port="2049"/>
/usr/lib/firewalld/services/ 下的服務

共有這些服務，可以直接叫用
amanda-client.xml high-availability.xml kpasswd.xml mysql.xml pop3s.xml smtp.xml
bacula-client.xml https.xml ldaps.xml nfs.xml postgresql.xml ssh.xml
bacula.xml http.xml ldap.xml ntp.xml proxy-dhcp.xml telnet.xml
dhcpv6-client.xml imaps.xml libvirt-tls.xml openvpn.xml radius.xml tftp-client.xml
dhcpv6.xml ipp-client.xml libvirt.xml pmcd.xml RH-Satellite-6.xml tftp.xml
dhcp.xml ipp.xml mdns.xml pmproxy.xml rpc-bind.xml transmission-client.xml
dns.xml ipsec.xml mountd.xml pmwebapis.xml samba-client.xml vnc-server.xml
ftp.xml kerberos.xml ms-wbt.xml pmwebapi.xml samba.xml wbem-https.xm

所謂的 zone 就表示主機位於何種環境，需要設定哪些規則，在 firewalld 裡共有 7 個zones
> 先決定主機要設定在那個區域 zone >> 再往該 zone 設定規則 >>> 重新讀取設定檔 sudo firewall-cmd --reload

public： 公開的場所，不信任網域內所有連線，只有被允許的連線才能進入，一般只要設定這裡就可以了

dmz： (Demilitarized Zone) 非軍事區，允許對外連線，內部網路只有允許的才可以連線進來


## 服务
firewall-cmd --reload

## 常用
* 顯示目前的設定
 firewall-cmd --list-all
 
* 關掉 DHCP 服務 port
firewall-cmd --zone=public --remove-service dhcpv6-client

* 暫時開啟 DNS port 53
# sudo systemctl start named
# sudo systemctl enable named
# sudo firewall-cmd --add-service=dns
# sudo firewall-cmd --reload
# firewall-cmd --list-all

* 永久開啟 DNS port 53
# sudo firewall-cmd --add-service=dns --permanent
# sudo firewall-cmd --reload

* 如何修改主機的預設 zone
vi /etc/firewalld/firewalld.conf
> 修改 DefaultZone=public
> 變成 DefaultZone=dmz
firewall-cmd --reload

* 加入自行指定的連接
 firewall-cmd --add-port=8080/tcp --permanent
success
# sudo firewall-cmd --reload
success
# sudo firewall-cmd --list-all

* 限制某服務只能從哪IP連入
# sudo firewall-cmd --add-rich-rule="rule family="ipv4" source address="192.168.1.88" service name="ssh" accept"
或 ip subnet
# sudo firewall-cmd --add-rich-rule="rule family="ipv4" source address="192.168.1.0/24" service name="ssh" accept"

* 限制某 port 只能從哪IP連入
firewall-cmd --add-rich-rule="rule family="ipv4" source address="192.168.12.9" port port="8080" protocol="tcp" accept"
注意：port port="8080" 不是寫錯唷！這是他的格式
* 從 zone 移除某項服務
# sudo firewall-cmd --zone=public --add-service=http --permanent
# sudo firewall-cmd --zone=public --remove-service=http --permanent

# sudo firewall-cmd --zone=public --add-port=8080/tcp --permanent
# sudo firewall-cmd --zone=public --remove-port=8080/tcp --permanent

## 参考资料
[CentOS 7 Firewalld 防火牆說明介紹](http://blog.xuite.net/tolarku/blog/363801991-CentOS+7+Firewalld+%E9%98%B2%E7%81%AB%E7%89%86%E8%AA%AA%E6%98%8E%E4%BB%8B%E7%B4%B9)



title: keepalived + nginx 爽活热备负载均衡
date: 2018-02-27 10:48:42
tags: [nginx, keepalived, 高性能,负载均衡]
categories: server
---
## 主要内容

keepalived + nginx 爽活热备负载均衡

*更新历史*
无

<!-- more -->

## 系统安装
CentOS 7.3
记得安装开发工具， 如果忘记可以在终端执行 `yum groupinstall "Development Tools"`

### ip config
设置静态IP， 主要根据实际情况设置最后6行。
```bash
# vim /etc/sysconfig/network-scripts/ifcfg-enp2s0 # 具体文件看自己服务器情况

TYPE=Ethernet

DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_PEERDNS=yes
IPV6_PEERROUTES=yes
NAME=enp2s0
UUID=d776619b-ac07-4367-b44c-40bd05aaea58
DEVICE=enp2s0

BOOTPROTO=static
ONBOOT=yes
DNS1=221.228.255.1
IPADDR=192.168.11.102
PREFIX=24
GATEWAY=192.168.11.1
```

设置完毕重启网络 `systemctl restart network.service `


### close SELINUX
```bash
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
systemctl stop firewalld
systemctl disable firewalld
```

### hostname
```bash
hostnamectl set-hostname `your hostname`
reboot
```
  修改 /etc/hosts 将 your hostname 写入到 ipv4 和 ipv6

## 设置YUM源
### 网络YUM源
+ 阿里CentOS 镜像
```bash
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo 
yum makecache #生成缓存
```

+ nginx
```bash
# vim /etc/yum.repos.d/nginx.repo
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=0
enabled=1
```

### 本地YUM源
[参照](http://geosmart.github.io/2015/12/07/CentOS%E9%85%8D%E7%BD%AE%E6%9C%AC%E5%9C%B0Yum%E6%BA%90/)

+ 编辑CentOS-Media.repo
其他的repo 都可改名成 ×××.repo.bak
拷贝 CentOS光盘 所有内容（比如路径 `/run/media/${user}/CentOS\ 7\ x86_64/`） 到 `/mnt/CentOS7_ISO/`

```bash
# vim /etc/yum.repos.d/CentOS-Media.repo

[local-media]
  name=CentOS-$releasever - Media
  baseurl=file:///mnt/CentOS7_ISO/
  gpgcheck=0
  enabled=1 #启动本地
  gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

```

## nginx
[nginx](http://giveme5.cc/2016/01/22/web%20server/nginx/nginx/)


## keepalive
### install
```bash
yum install -y keepalived
```

### config
假设主服务器IP:192.168.0.201,从服务器ip:192.168.0.202 虚拟ip:192.168.0.200

#### master
`vim /etc/keepalived/keepalived.conf`

```bash
! Configuration File for keepalived

global_defs {
#   notification_email {
#     acassen@firewall.loc
#     failover@firewall.loc
#     sysadmin@firewall.loc
#   }
#   notification_email_from Alexandre.Cassen@firewall.loc
#   smtp_server 192.168.200.1
#   smtp_connect_timeout 30
   router_id nginx_master      #机器标识， 通常是hostname 
#   vrrp_skip_check_adv_addr
#   vrrp_strict
   vrrp_garp_interval 0
   vrrp_gna_interval 0
}

vrrp_script chk_nginx {  #    script "killall -0 nginx"
    script "/etc/keepalived/check_ng.sh"        #检测nginx的脚本, 脚本名称千万不要使用 完整 nginx，容易出错
    interval 2                                  #每2秒检测一次
    weight -5                                   #如果某一个nginx宕机 则权重减5
    fall 3  
    rise 2
}

vrrp_instance VI_1 {
    state MASTER            #状态 MASTER BACKUP
    interface enp0s3        #实例绑定的网卡，因为在配置虚拟IP的时候必须是在已有的网卡上添加的
    virtual_router_id 51    #虚拟路由的ID号,两个节点设置必须一样
    priority 101            #主备优先级
    advert_int 1
    
    # 设置验证信息，两个节点必须一致
    authentication {
        auth_type PASS      #认证方式
        auth_pass 1111      #明文密码
    }
    virtual_ipaddress {
        192.168.0.200
    }
    track_script {
       chk_nginx
    }
}
```


#### track_script
脚本名称千万不要使用 完整 `nginx`，容易出错
`/etc/keepalived/check_ng.sh`
记得加上执行权限

```bash
#!/bin/bash

A=`pgrep nginx|wc -l`
if [ $A -eq 0 ];then

  #yum install
 /bin/systemctl start nginx.service
 
 # 源码安装 nginx
 # /usr/local/nginx/sbin/nginx

 if [ `pgrep nginx|wc -l` -eq 0 ];then
   /bin/systemctl stop keepalived.service
 fi
fi
```

也可以根据自己的业务需求，总结出在什么情形下关闭keepalived。
如 curl 主页连续2个3s没有响应则切换：

```bash
#!/bin/bash
count=0
for (( k=0; k<2; k++ ))
do
    check_code=$( curl --connect-timeout 3 -sL -w "%{http_code}\\n" http://localhost/ -o /dev/null )
    if [ "$check_code" != "200" ]; then
        count=$(expr $count + 1)
        sleep 3
        continue
    else
        count=0
        break
    fi
done
if [ "$count" != "0" ]; then
    exit 1
else
    exit 0
fi
```

#### backup
在其它备机BACKUP上，只需要改变 state MASTER -> state BACKUP，priority 101 -> priority 100即可
`vim /etc/keepalived/keepalived.conf`

```bash
! Configuration File for keepalived

global_defs {
#   notification_email {
#     acassen@firewall.loc
#     failover@firewall.loc
#     sysadmin@firewall.loc
#   }
#   notification_email_from Alexandre.Cassen@firewall.loc
#   smtp_server 192.168.200.1
#   smtp_connect_timeout 30
   router_id nginx_backup      #机器标识， 通常是hostname 
#   vrrp_skip_check_adv_addr
#   vrrp_strict
   vrrp_garp_interval 0
   vrrp_gna_interval 0
}

vrrp_script chk_nginx {
#    script "killall -0 nginx"
    script "/etc/keepalived/check_ng.sh"     #检测nginx的脚本
    interval 2                                  #每2秒检测一次
    weight -5                                   #如果某一个nginx宕机 则权重减5
    fall 3  
    rise 2
}

vrrp_instance VI_1 {
    state BACKUP            #状态 MASTER BACKUP
    interface enp0s3        #实例绑定的网卡，因为在配置虚拟IP的时候必须是在已有的网卡上添加的
    virtual_router_id 51    #虚拟路由的ID号,两个节点设置必须一样
    priority 100            #主备优先级
    advert_int 1
    
    # 设置验证信息，两个节点必须一致
    authentication {
        auth_type PASS      #认证方式
        auth_pass 1111      #明文密码
    }
    virtual_ipaddress {
        192.168.0.200
    }
    track_script {
       chk_nginx
    }
}
```

##附录
### keepalived 配置选项说明
+ `global_defs`

`notification_email` ： keepalived在发生诸如切换操作时需要发送email通知地址，后面的 smtp_server 相比也都知道是邮件服务器地址。也可以通过其它方式报警，毕竟邮件不是实时通知的。
`router_id` ： 机器标识，通常可设为hostname。故障发生时，邮件通知会用到

+ `vrrp_instance`

state ： 指定instance(Initial)的初始状态，就是说在配置好后，这台服务器的初始状态就是这里指定的，但这里指定的不算，还是得要通过竞选通过优先级来确定。如果这里设置为MASTER，但如若他的优先级不及另外一台，那么这台在发送通告时，会发送自己的优先级，另外一台发现优先级不如自己的高，那么他会就回抢占为MASTER
interface ： 实例绑定的网卡，因为在配置虚拟IP的时候必须是在已有的网卡上添加的
mcast_src_ip ： 发送多播数据包时的源IP地址，这里注意了，这里实际上就是在那个地址上发送VRRP通告，这个非常重要，一定要选择稳定的网卡端口来发送，这里相当于heartbeat的心跳端口，如果没有设置那么就用默认的绑定的网卡的IP，也就是interface指定的IP地址
virtual_router_id ： 这里设置VRID，这里非常重要，相同的VRID为一个组，他将决定多播的MAC地址
priority ： 设置本节点的优先级，优先级高的为master
advert_int ： 检查间隔，默认为1秒。这就是VRRP的定时器，MASTER每隔这样一个时间间隔，就会发送一个advertisement报文以通知组内其他路由器自己工作正常
authentication ： 定义认证方式和密码，主从必须一样
virtual_ipaddress ： 这里设置的就是VIP，也就是虚拟IP地址，他随着state的变化而增加删除，当state为master的时候就添加，当state为backup的时候删除，这里主要是有优先级来决定的，和state设置的值没有多大关系，这里可以设置多个IP地址
track_script ： 引用VRRP脚本，即在 vrrp_script 部分指定的名字。定期运行它们来改变优先级，并最终引发主备切换。

+ `vrrp_script`

告诉 keepalived 在什么情况下切换，所以尤为重要。可以有多个 vrrp_script

script ： 自己写的检测脚本。也可以是一行命令如killall -0 nginx
interval 2 ： 每2s检测一次
weight -5 ： 检测失败（脚本返回非0）则优先级 -5
fall 2 ： 检测连续 2 次失败才算确定是真失败。会用weight减少优先级（1-255之间）
rise 1 ： 检测 1 次成功就算成功。但不修改优先级
这里要提示一下script一般有2种写法：

通过脚本执行的返回结果，改变优先级，keepalived继续发送通告消息，backup比较优先级再决定
脚本里面检测到异常，直接关闭keepalived进程，backup机器接收不到advertisement会抢占IP
上文 vrrp_script 配置部分，killall -0 nginx属于第1种情况，/etc/keepalived/check_ng.sh属于第2种情况（脚本中关闭keepalived）。个人更倾向于通过shell脚本判断，但有异常时exit 1，正常退出exit 0，然后keepalived根据动态调整的 vrrp_instance 优先级选举决定是否抢占VIP：

如果脚本执行结果为0，并且weight配置的值大于0，则优先级相应的增加
如果脚本执行结果非0，并且weight配置的值小于0，则优先级相应的减少
其他情况，原本配置的优先级不变，即配置文件中priority对应的值。

提示：

优先级不会不断的提高或者降低
可以编写多个检测脚本并为每个检测脚本设置不同的weight（在配置中列出就行）
不管提高优先级还是降低优先级，最终优先级的范围是在[1,254]，不会出现优先级小于等于0或者优先级大于等于255的情况
在MASTER节点的 vrrp_instance 中 配置 nopreempt ，当它异常恢复后，即使它 prio 更高也不会抢占，这样可以避免正常情况下做无谓的切换
以上可以做到利用脚本检测业务进程的状态，并动态调整优先级从而实现主备切换。

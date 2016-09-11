title: 1-23 RHEL7启动配置
date: 2015-12-24 16:29:44
tags: GOD linux
categories: linux
---

## 主要内容
* 系统启动配置
* Grub 加固
* 服务启动管理

<!-- more -->


## 系统启动配置
RHEL7 依然可以通过MBR启动，MBR保存着系统的主引导程序（grub 446字节，分区表64字节，2字节校验），启动过程就是把内核加载到内存。

### 启动的顺序：
1、BIOS；
2、BIOS激活MBR；
3、MBR中的引导程序(grub)加载到内存，生成一个微系统（如xfs）；
4、grub 读取分区表，找到引导分区；
5、grub读取自身的配置文件，找到内核文件
6、加载内核文件；
 

RHEL7中第一个启动进程是 `systemd`
```bash
[root@localhost ~]#pstree -p | more
systemd(1)-+-ModemManager(1017)-+-{ModemManager}(1049)
           |                    `-{ModemManager}(1066)
           |-NetworkManager(1006)-+-{NetworkManager}(1079)
           |                      |-{NetworkManager}(1081)
```

### RHEL7设置启动级别
RHEL7 不在使用 `inittab` ， 打开`inittab` 可以看到如下内容;
```bash
[root@localhost ~]# vim /etc/inittab
inittab is no longer usedwhen using systemd.
#当使用systemd inittab不再使用。
# ADDING CONFIGURATIONHERE WILL HAVE NO EFFECT ON YOUR SYSTEM.
#
# Ctrl-Alt-Delete ishandled by /usr/lib/systemd/system/ctrl-alt-del.target
#
# systemd uses 'targets'instead of runlevels. By default, there are two main targets:
# systemd使用“目标”而不是运行级。默认情况下,有两个主要目标
# multi-user.target:analogous to runlevel 3
# graphical.target:analogous to runlevel 5
#两种运行级别分别对应第三运行级别和第五运行级别
# To view current defaulttarget, run:
# systemctl get-default
#查看当前运行级别
# To set a defaulttarget, run:
# systemctl set-defaultTARGET.target
#设置运行级别
```


#### 查看当前运行级别
```bash
[root@localhost ~]# systemctlget-default
graphical.target
```

#### 切换运行级别
```bash
[root@localhost ~]# systemctl isolate graphical.target
```

#### 设置默认的运行级别
```bash
[root@localhost ~]# systemctl set-default multi-user.target
rm'/etc/systemd/system/default.target'
ln -s '/usr/lib/systemd/system/multi-user.target' '/etc/systemd/system/default.target'
```

### grub引导配置
* 主要配置文件

`/etc/grub2.cfg` `/etc/default/grub`

```bash
[root@localhost ~]# ll /etc/grub2.cfg
lrwxrwxrwx. 1 root root 22 Nov22 03:40 /etc/grub2.cfg -> ../boot/grub2/grub.cfg
  
[root@localhost ~]# ll /etc/default/grub
-rw-r--r--. 1 root root 192Nov 22 03:46 /etc/default/grub
```

* 修改配置 
修改系统启动参数的时候，不要直接修改 ，因为如果后期更新内核的时候，那个grub.cfg也会自动更新，先前所做的配置会全部失效，如果需要修改，建议修改/etc/default/grub，然后使用grub2-mkconfig 命令生效。这个文件是由/etc/grub.d/00_header文件调用

```bash
[root@localhost ~]# vim /etc/default/grub
  
#选择菜单的显示时间，默认是5，值是0表示不显示菜单选项，值是-1表示无限期的等待，直到用户做出选择
GRUB_TIMEOUT=5 
  
#设置默认启动项，按menuentry顺序。比如要默认从第四个菜单项启动，数字改为3，若#改为 saved，#则默认为上次启动项。
GRUB_DEFAULT=saved 
  
#修改完成之后通过grub2-mkconfig命令重新生成内核参数
[root@localhost ~]#grub2-mkconfig 
```




## Grub 加固
### 明文
```bash
[root@localhost ~]# vim /etc/grub.d/00_header
  
#末尾增加如下内容
cat <<EOF
set superusers="sam"
password sam 123456
EOF
  
#更新grub
[root@localhost ~]#grub2-mkconfig -o /boot/grub2/grub.cfg
```
注意：用来加密的用户sam和系统中的用户没有任何关系， sam不是系统用户都可以。

### pbkdfv2算法加密
grub1.98版之后，可以设定加密的密码

* 生成 pbkdfv2 加密的密码
```bash
[root@localhost Desktop]# grub2-mkpasswd-pbkdf2 
Enter password:   #输入 123
Reenter password:  #输入 123
PBKDF2 hash of your password is grub.pbkdf2.sha512.10000.8EE07022D712A9EDB5EA8CA2AA8FC8B3166D903FC2BD058FAE95F8950D115EE8722099F07EBB223D0B9475B90B01D8A6C04580273B0866674005603554AFEF0C.28A856AC620C831BC1E04C2E23B5602BEDCC6910EBFCA3D2ADEFE3F39166AAFE7DAE4EFF10E090945009DE73D2D3F0005E32930F13F8D55CD7F34ACF9C77276F
```

* 将加密的密码写入/etc/grub.d/00_header文件
```bash
[root@localhost ~]# vim /etc/grub.d/00_header
  
#末尾增加如下内容
cat <<EOF
set superusers="sam"
password_pbkdf2  sam grub.pbkdf2.sha512.10000.8EE07022D712A9EDB5EA8CA2AA8FC8B3166D903FC2BD058FAE95F8950D115EE8722099F07EBB223D0B9475B90B01D8A6C04580273B0866674005603554AFEF0C.28A856AC620C831BC1E04C2E23B5602BEDCC6910EBFCA3D2ADEFE3F39166AAFE7DAE4EFF10E090945009DE73D2D3F0005E32930F13F8D55CD7F34ACF9C77276F
EOF
```

* 更新grub
```bash
[root@localhost ~]# grub2-mkconfig -o /boot/grub2/grub.cfg
```

验证：重启， 在grub页面 输入 `e`； 用户 `sam`； 密码 `123`

## 服务启动管理
### 服务启动配置
RHEL7中所有的开机启动的服务，都会在 `/etc/systemd/system/multi-user.target.wants` 中有对应的链接文件
将服务设置为开机自动启动，其实就是将 `/usr/lib/systemd/system` 目录下的服务配置文件做一个软链接到/etc/systemd/system/multi-user.target.wants/目录下

* 开机启动
例：安装httpd服务，并设置为开机自动启动
```bash
[root@localhost ~]# yum -y install httpd
  
[root@localhost ~]# systemctl enable httpd
ln -s '/usr/lib/systemd/system/httpd.service''/etc/systemd/system/multi-user.target.wants/httpd.service'
```
 
* 开机不启动
将服务设置为自动关闭，其实就是将/usr/lib/systemd/system/目录下的服务配置文件的软链接进行删除
```bash
[root@localhost ~]# systemctl disable httpd
rm'/etc/systemd/system/multi-user.target.wants/httpd.service'
```

* 查看服务开机启动状态
```bash
[root@localhost ~]# systemctlis -enabled httpd
disabled
```

* 启动和关闭服务
```bash
[root@localhost ~]# systemctl restart httpd
  
[root@localhost ~]# systemctl stop  httpd
```

### Systemd && systemV
Systemd和systemV服务启动方式对比
1）systemV 在服务启动方面采用的是顺序启动，即每一个服务都有对应的启动顺序，优先级越高那么在服务启动时就会被优先启动
2）systemd    在服务启动方面则是采用了并行启动的方式，而且按需启动，减少系统资源消耗，大大节省了系统启动的等待时间
 
相比较systemd而言，systemv的优势
1）原理简单，易于理解
2）依靠shell脚本控制，编写服务脚本门槛比较低

缺点：
1）服务顺序启动，启动过程比较慢
2）不能做到根据需要来启动服务，比如：通常希望在插入U盘的时候，再启动USB控制的服务，这样可以更好的节省系统资源
 
原本，systemv的服务启动慢，并不是一个问题，尤其是Linux系统以前主要是运行在服务器上，常年也难得重启一次，每次重启硬件检测都需要5分钟以上，相对来说系统启动已经很快了
但是随着移动互联网的到来，Systemv 服务启动慢的问题显得越来越突出，许多移动设备都是基于Linux内核，系统启动比较频繁，如果每次启动时都要等待服务顺序启动，显然难以接受，systemd就是为了解决这个问题诞生的

### 其他
#### 查看系统启动时间
```bash
[root@localhost ~]#systemd-analyze
Startup finished in6.894s (kernel) + 2.939s (initrd) + 12.978s (userspace) = 22.812s
```

#### 查看每个服务的启动时间
```bash
[root@localhost Desktop]# systemd-analyze blame 
           405ms network.service
           350ms postfix.service
           320ms lvm2-monitor.service
           279ms plymouth-quit-wait.service
           255ms tuned.service
           130ms NetworkManager.service
           118ms ModemManager.service
           107ms boot.mount
            95ms accounts-daemon.service
            79ms avahi-daemon.service
            76ms chronyd.service
            75ms rhel-dmesg.service
            70ms abrt-ccpp.service
            70ms rsyslog.service
            61ms ksm.service
            56ms systemd-logind.service
            47ms systemd-vconsole-setup.service
            45ms rtkit-daemon.service
            43ms sysstat.service
            42ms libvirtd.service
            42ms auditd.service
            40ms plymouth-read-write.service
            37ms polkit.service

```

#### 查看严重消耗时间的服务树状表
```bash
[root@localhost ~]# systemd-analyze critical-chain
The time after the unitis active or started is printed after the "@" character.
The time the unit takesto start is printed after the "+" character.
graphical.target @12.973s
└─multi-user.target@12.972s
  └─postfix.service @10.044s +2.070s
    └─network.target @10.020s
      └─network.service @9.218s +801ms
        └─NetworkManager.service @5.186s +4.031s
          └─basic.target @5.168s
```

#### 列出所有服务并且检查是否开机启动
```bash
[root@localhost ~]# systemctl list-unit-files --type service
UNIT FILE                                   STATE
abrt-ccpp.service                           enabled
abrt-oops.service                           enabled
abrt-pstoreoops.service                     disabled
abrt-vmcore.service                         enabled
abrt-xorg.service                           enabled
```
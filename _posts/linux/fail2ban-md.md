---
title: fail2ban防御暴力破解攻击
date: 2017-03-28 17:30:41
tags: [linux, fail2ban, 安全]
categories: linux
---

主要内容

* 安装
* 配置
* 使用

使用 fail2ban 防御服务器的暴力破解攻击， ssh，nginx 服务为例。

<!-- more -->

实验环境 
```sh
╰─# lsb_release -a
LSB Version:    :core-4.1-amd64:core-4.1-noarch
Distributor ID: CentOS
Description:    CentOS Linux release 7.2.1511 (Core) 
Release:        7.2.1511
Codename:       Core

```

## 安装
### EPEL 源
需要安装 EPEL 源， 国内可使用阿里云的镜像。

```sh
mv /etc/yum.repos.d/epel.repo /etc/yum.repos.d/epel.repo.backup
wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
```
### fail2ban

```sh
yum install fail2ban-firewalld
```

## 配置
配置文件根路径 `/etc/fail2ban/`。 默认服务的配置文件 `jail.conf`。 
自定义服务配置文件 存放路径 `jail.d`， 建议以 `jail_服务名.local` 命名。
各个服务的拦截条件 在 `filter.d` 。

## ssh服务配置

```sh
╰─# vim jail.d/jail_ssh.local 

```

增加如下内容 
```sh
[sshd]
enabled = true
maxretry = 3 # 尝试次数
bantime = 7200 # 禁止访问时间（秒）

```

如果你使用用户密码登录，以上配置就可以了。

如果你使用公钥登录， 在修改下 ssh 的拦截条件, 增加 `^%(__prefix_line)sConnection closed by <HOST> \[preauth\]$`

```sh
╰─# vim filter.d/sshd.conf 
```

```sh
# PasswordAuthentication in sshd_config.
#
#
# "Connection from <HOST> port \d+" requires LogLevel VERBOSE in sshd_config
#

[INCLUDES]

# Read common prefixes. If any customizations available -- read them from
# common.local
before = common.conf

[Definition]

_daemon = sshd

failregex = ^%(__prefix_line)s(?:error: PAM: )?[aA]uthentication (?:failure|error|failed) for .* from <HOST>( via \S+)?\s*$
            ^%(__prefix_line)s(?:error: PAM: )?User not known to the underlying authentication module for .* from <HOST>\s*$
            ^%(__prefix_line)sFailed \S+ for (?P<cond_inv>invalid user )?(?P<user>(?P<cond_user>\S+)|(?(cond_inv)(?:(?! from ).)*?|[^:]+)) from <HOST>(?: port \d+)?(?: ssh\d*)?(?(cond_user):|(?:(?:(?! from ).)*)$)
            ^%(__prefix_line)sROOT LOGIN REFUSED.* FROM <HOST>\s*$
            ^%(__prefix_line)s[iI](?:llegal|nvalid) user .*? from <HOST>(?: port \d+)?\s*$
            ^%(__prefix_line)sUser .+ from <HOST> not allowed because not listed in AllowUsers\s*$
            ^%(__prefix_line)sUser .+ from <HOST> not allowed because listed in DenyUsers\s*$
            ^%(__prefix_line)sUser .+ from <HOST> not allowed because not in any group\s*$
            ^%(__prefix_line)srefused connect from \S+ \(<HOST>\)\s*$
            ^%(__prefix_line)s(?:error: )?Received disconnect from <HOST>: 3: .*: Auth fail(?: \[preauth\])?$
            ^%(__prefix_line)sUser .+ from <HOST> not allowed because a group is listed in DenyGroups\s*$
            ^%(__prefix_line)sUser .+ from <HOST> not allowed because none of user's groups are listed in AllowGroups\s*$
            ^(?P<__prefix>%(__prefix_line)s)User .+ not allowed because account is locked<SKIPLINES>(?P=__prefix)(?:error: )?Received disconnect from <HOST>: 11: .+ \[preauth\]$
            ^(?P<__prefix>%(__prefix_line)s)Disconnecting: Too many authentication failures for .+? \[preauth\]<SKIPLINES>(?P=__prefix)(?:error: )?Connection closed by <HOST> \[preauth\]$
            ^(?P<__prefix>%(__prefix_line)s)Connection from <HOST> port \d+(?: on \S+ port \d+)?<SKIPLINES>(?P=__prefix)Disconnecting: Too many authentication failures for .+? \[preauth\]$
            ^%(__prefix_line)s(error: )?maximum authentication attempts exceeded for .* from <HOST>(?: port \d*)?(?: ssh\d*)? \[preauth\]$
            ^%(__prefix_line)spam_unix\(sshd:auth\):\s+authentication failure;\s*logname=\S*\s*uid=\d*\s*euid=\d*\s*tty=\S*\s*ruser=\S*\s*rhost=<HOST>\s.*$
            ^%(__prefix_line)sConnection closed by <HOST> \[preauth\]$  ##增加在这里

ignoreregex =

[Init]

# "maxlines" is number of log lines to buffer for multi-line regex searches
maxlines = 10

journalmatch = _SYSTEMD_UNIT=sshd.service + _COMM=sshd

# DEV Notes:
#
#   "Failed \S+ for .*? from <HOST>..." failregex uses non-greedy catch-all because
#   it is coming before use of <HOST> which is not hard-anchored at the end as well,
#   and later catch-all's could contain user-provided input, which need to be greedily
#   matched away first.
#
# Author: Cyril Jaquier, Yaroslav Halchenko, Petr Voralek, Daniel Black
```

## 自定义服务
以下检查 nginx 服务 400,404,500 错误状态为例



### jail
路径 `/etc/fail2ban/jail.d` , 新建 `jail_nginx-err.local`
内容如下 
```sh
[nginx-err]
enabled = true
port = http,https
filter = nginx-err
banaction = firewallcmd-ipset
logpath = /var/log/nginx/access.log
findtime = 600
maxretry = 20
bantime = 7200
```

大致意义如下：

+ [nginx-err]：定义jail名称
+ enabled：是否启用该jail，默认的所有规则都没有该项，需要手动添加
+ port：指定封禁的端口，默认为0:65535，也就是所有端口，但可以在jail中设定
+ filter：指定过滤器名称
+ logpath：日志路径
+ banaction：达到阈值后的动作， 默认
+ maxretry：阈值
+ findtime：时间间隔 单位秒
+ bantime：封禁时长
+ ignoreip：忽略的IP

在这里有几点要注意的：

+ logpath与banaction可以有多行，如action中的设定：
++ 调用firewallcmd-ipset封禁目标IP访问的多个端口
++ 调用sendmail发送告警邮件
+ findtime不是检查日志的时间间隔，日志的检查是实时的。因为fail2ban自带数据库，所以可以在设定的时间内统计匹配次数
+ ignoreip添加后端服务器的IP或CDN的IP


### action
使用现有的 `firewallcmd-ipset`

### filter
路径 `/etc/fail2ban/filter.d` 新建 `nginx-err.conf`


```sh
[Definition]

failregex = ^<HOST> -.*- .*HTTP/1.* (404|400|500) \d+ .*$

ignoreregex =

```


注意：
nginx log 格式
```
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
```

## 使用
* 启动/重启 
```
╰─# systemctl restart fail2ban.service
```

* 开机启动
```
╰─# systemctl enable fail2ban.service
```

* 检验fail2ban状态， 显示出当前活动的监狱列表
```sh
╰─# fail2ban-client status                                                                                                                                                                                  255 ↵
Status
|- Number of jail:      1
`- Jail list:   sshd
```

* 检验一个特定监狱的状态
```sh
╰─# fail2ban-client status sshd 
Status for the jail: sshd
|- Filter
|  |- Currently failed: 1
|  |- Total failed:     4
|  `- Journal matches:  _SYSTEMD_UNIT=sshd.service + _COMM=sshd
`- Actions
   |- Currently banned: 1
   |- Total banned:     2
   `- Banned IP list:   140.205.201.42
```

* 解锁特定的IP地址
```sh
# fail2ban-client set 特定监狱 unbanip ip
fail2ban-client set sshd unbanip 192.168.1.8
```

* 查看防火墙
```sh
firewall-cmd --direct --get-all-rules
  
ipset list
```




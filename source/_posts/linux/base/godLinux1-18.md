title: 1-18-计划任务和日志管理
date: 2015-12-18 15:29:44
tags: GOD linux
categories: linux
---
## 主要内容
* [at 定制单次执行的计划任务](./#at)
* [cron 定制周期性计划任务](./#crontab)
* [配置服务器自动开关机](./#服务器自动开机)
* [Linux系统日志记录规律](./#日志记录方式)
* [自定义日志记录方式](./#自定义Linux日志)
* [配置远程收集日志服务器](./#远程日志服务)

<!-- more -->

## at
* 作用： 定制单次执行的计划任务
* 语法： at $时间

### 前置条件
服务 `atd` 必须开启
```bash
[root@localhost ~]#systemctl status atd # 查看 atd 状态
[root@localhost ~]#systemctl restart atd #重启atd服务
[root@localhost ~]#systemctl enable atd #设置atd服务开机自动启动
```

### 创建
```bash
[root@localhost ~]# at 20:39
at> echo `date`  > /root/date.txt      #输入要执行的命令
at> <EOT>                              #按ctrl+D结束
job 1 at Thu Dec 17 20:39:00 2015
```
也可以这样写
```bash
[root@localhost ~]# at 20:00 2015-12-18
[root@localhost ~]# at now + 10min
```

### 查看
```
[root@localhost ~]# at -l
```

创建成功的at计划任务会在这个目录下成生一个可执行的脚本文件：
```bash
[root@localhost ~]# ll /var/spool/at/*
-rwx------  1 root  root   4325 Dec 17 20:43 /var/spool/at/a000020170ddb0
```

### at任务删除：
atrm  at序列号( at -l 查看)
```
[root@localhost ~]#atrm 3
```

## crontab
定制周期性计划任务

### 语法
crontab [-u username] [-e / -l / -r] 
* -u username : 指定用户， 默认当前用户
* -e : edit 编辑，新增
* -l : list 显示
* -r ：删除

### 前提
crond服务已启动
```bash
[root@localhost ~]#systemctl status crond # 查看
crond.service - CommandScheduler
   Loaded: loaded(/usr/lib/systemd/system/crond.service; enabled)
   Active: active (running) since Thu2015-12-17 20:24:06 CST; 30min ago
  
[root@localhost ~]#systemctl restart crond        #启动
[root@localhost ~]#systemctl enable crond         #设置开机自动启动
```

### 新建/编辑
`crontab –e` 后编辑， vi环境

* 语法
格式：分 时 日 月 星 谁做后面的事情 命令
![](http://7xklqw.com1.z0.glb.clouddn.com/cron.png)

* 取值范围
分：0－59
小时：0－23
日：1－31
月：1－12
周：0－7   0 7 都是周日
  
取值为 `*` 时表示该单位无限制
取值为 `a-b` 时表示从第 a 到第 b 这段时间内要执行
取值为 `*/n` 时表示每 n 个时间间隔执行一次
取值为 `a, b, c,...` 表示第 a, b, c,... 时间要执行

* 例
```
 #每月9,18,22号这几天的凌晨1点1分，执行一个备份脚本
1 1 9,18,22 *  * ./backup.sh
  
 #每月9-22号这几天的凌晨1点1分，执行一个备份脚本
1 1 9-22 * *    ./backup.sh
 
 #每5分钟，执行一次
*/5  * *  *  *  ./backup.sh
```

### 系统级别的计划任务，
需要执行的命令和脚本都放在这里：

```bash
[root@localhost ~]# ls /etc/cron.*

/etc/cron.hourly   /etc/cron.daily   /etc/cron.weekly    /etc/cron.monthly
```

### anacron
cron用控制循环执行例行性工作。如果我要设定机器每早8点进行备份用服务。除非我机器保证在8点这个时间点不会关机，如果关机了，cron中的脚本，在下次开机将不会被执行。

anacron并没有取代cron的意思，anacron用于，机器重启后，会侦测停机期间，有没有cron没有执行的计划任务，如果有，会立即，执行一下没有执行的任务。


## 服务器自动开机
通过BIOS设置，可以实现定时开机。
* 开机后出现主板画面是按Delete这个键，部分品牌机可能按F2，F1， 进入BIOS设置。
* Power Management Setup，就进入电源管理设置。
* Wake Up Event Setup （如果没有，代表不支持自动开机）
* Resume By RTC Alarm， 将Disabied 更改为Enabled，然后继续回车确定。然后再继续设置时间点和日期。
假如你需要每天都定时开机，就选择Every Day，，你如果想要在每天6:45开机，就通过数字键输入06：45:00，最后，一般按F10 进行保存，重启电脑后生效！
![](http://7xklqw.com1.z0.glb.clouddn.com/resumertc.png)


## 日志管理
### 常用的系统日志
下列为常用的系统日志，无特殊说明的可以按文本方式查看。
* 核心启动日志:            /var/log/dmesg
* 系统报错或重启服务等日志:  /var/log/messages
* 邮件系统日志:            /var/log/maillog
* cron(定制任务日志)日志:   /var/log/cron   #计划日志执行成功与否，在这个文件中看
* 验证系统用户登录          /var/log/secure 
* 记录所有的登入和登出:      /var/log/wtmp 
`last` 命令查看
```bash
[root@localhost ~]# last
wtmp begins Thu Dec 1721:44:40 2015
```
* 记录每個用戶最后的登入信息: /var/log/lastlog 
`lastlog` 命令查看
```bash
[root@localhost ~]# lastlog
Username         Port     From             Latest
root             pts/2   192.168.1.101    Thu Dec 1721:00:38 +0800 2015
```
* 记录错误的登入尝试:        /var/log/btmp 
`lastb` 命令查看
```bash
[root@localhost ~]#lastb
mk       ssh:notty    192.168.1.69     Thu Dec 17 21:48 - 21:48  (00:00)   
mk       ssh:notty    192.168.1.69     Thu Dec 17 21:48 - 21:48  (00:00)   
```
  注 `如果btmp文件特别大，说明有人在暴力破解你的服务器`
  ```bash
  [root@localhost ~]# ll -h /var/log/btmp
  -rw-------. 1 root utmp1.2K Dec 17 21:48 /var/log/btmp
  ```

### 日志记录方式
Linux的日志记录方式： 先分类，然后每个类中再分级别
<br>
#### Linux日志分类
主要7种日志分类(FACILITY):

|分类|说明|
|-----------------|---------------------|
|`authpriv` |  安全认证相关|
|`cron`      | at和cron定时相关|
|`daemon`    | 后台进程相关|
|`kern`      | 内核产生|
|`lpr`       | 打印系统产生|
|`mail`      | 邮件系统相关|
|`syslog`    | 日志服务本身|
|`news`      | 新闻系统  （和BBS差不多，新闻组）|
|`uucp`      | uucp系统产生 。Unix-to-Unix Copy(UNIX至UNIX的拷贝)，Unix系统的一项功能，允许计算机之间以存储-转发方式交换e-mail和消息。在Internet兴起之前是Unix系统之间连网的主要方式。|
|`local0到local7` |共8个类型，系统保留的：8个系统日志类型，给其它程序使用。或用户 自定义用|
<br>
#### linux日志级别 
8个日志级别(PRIOROTY)：以下排列，由轻到重

|级别(PRIOROTY) |说明|
|--------------|---------------------------------|
|`debug` |                 排错信息。开发人|
|`info` |                  正常信息|
|`notice` |                稍微要注意的|
|`warn` |                  警告|
|`err(error)` |            错误|
|`crit(critical)` |         关键的错误|
|`alert` |                 警报警惕|
|`emerg(emergency)` |      紧急，突发事件|



<br><br>
### 日志服务管理
<br>
#### linux 日志配置文件

|系统|服务名称|配置文件|
|---------|-----------------|-----------------------|
|RHEL5|syslog|/etc/syslog.conf|
|RHEL6|rsyslog|/etc/rsyslog.conf|
|RHEL7|rsyslog|/etc/rsyslog.conf|


#### 配置文件内容摘要
```bash
 46 #### RULES ####
 47 
 48 # Log all kernel messages to the console.
 49 # Logging much else clutters up the screen.
 50 #kern.*                                                 /dev/console
 51 
 52 # Log anything (except mail) of level info or higher.
 53 # Don't log private authentication messages!
 54 *.info;mail.none;authpriv.none;cron.none                /var/log/messages
 55 
 56 # The authpriv file has restricted access.
 57 authpriv.*                                              /var/log/secure
 58 
 59 # Log all the mail messages in one place.
 60 mail.*                                                  -/var/log/maillog
 61 
 62 
 63 # Log cron stuff
 64 cron.*                                                  /var/log/cron
 65 
 66 # Everybody gets emergency messages
 67 *.emerg                                                 :omusrmsg:*
 68 
 69 # Save news errors of level crit and higher in a special file.
 70 uucp,news.crit                                          /var/log/spooler
 71 
 72 # Save boot messages also to boot.log
 73 local7.*                                                /var/log/boot.log
```
说明：
 `kern.*` : 内核类型的所级别日志
 `*.info;mail.none;news.none;authpriv.none;cron.none` : 由于 mail, news, authpriv, cron 等类别产生的讯息较多，因此在 /var/log/messages 里面不记录这些项目。除此其他讯息都写入/var/log/messages 中。所以messages 文件很重要
 `authpriv.*` : 认证方面的讯息均写入 /var/log/secure 档案；
 `mail.* -/var/log/maillog ` : 邮件方面的讯息则均写入 /var/log/maillog 档案； 减号`-`是干嘛用的？由于邮件所产生的讯息比较多，因此我们希望邮件产生的讯息先储存在速度较快的内存中 (buffer) ，等到数据量够大了才一次性的将所有数据都填入磁盘内，这样将有利于减少对磁盘读写的次数，减少IO读写开销。另外，由于讯息是暂存在内存内，因此若不正常关机导致登录信息未写入到文档中，可能会造成部分数据的遗失。
 `cron.*` : 例行性工作排程均写入 /var/log/cron 档案；
 `local7.*` : 将本机开机时应该显示到屏幕的讯息写入到 /var/log/boot.log 档案中；

<br>
#### 规则：
`.none` : 该类型所有的等级不记录
`. ` ：代表『比后面还要高的等级都被记录下来』的意思，例如： mail.info 代表只要是 mail 类型的信息，而且该信息等级高于 info (包括 info 本身)时，就会被记录下来的意思。
`.=`  ：代表所需要的等级就是后面接的等级而已， 其他的都不要！
`.!`  ：代表不等于，亦即是除了该等级外的其他等级都记录。

举例：
`cron.none`   对于cron类型日志不记录任何信息
`cron.=err`   对于cron类型日志只记录err级别的信息
`cron.err`    对于cron类型日志记录大于err级别的信息
`cron.!err`   对于cron类型日志不记录err级别的信息，其他级别都记录。
 <br>
#### 记录日志的位置：
* 日志的相对路径：通常就是放在 /var/log中
* 存在远程日志服务器上
* 有时日志会直接弹出在屏幕上。类似于wall命令。


### 自定义Linux日志
以sshd服务为例，说明自定义的日志
* 自定义local0日志
local7 已经被系统定义为boot日志，需要使用未被使用的分类
```bash
[root@localhost ~]# vim /etc/rsyslog.conf
local0.*                                                /var/log/sshd.log
[root@localhost ~]#systemctl restart rsyslog.service
```
  
* 配置sshd服务的配置文件
```
[root@localhost ~]# vim /etc/ssh/sshd_config
SyslogFacility local0
[root@localhost ~]#systemctl restart sshd
[root@localhost ~]# ls /var/log/sshd.log
/var/log/sshd.log
  
[root@localhost ~]# cat !$
cat /var/log/sshd.log
Dec 17 22:18:38localhost sshd[35876]: Server listening on 0.0.0.0 port 22.
Dec 17 22:18:38localhost sshd[35876]: Server listening on :: port 22.
```


 

### 日志回滚
日志回滚 logrotate（日志回滚过程： 创建新文件、改名旧文件。）

* 配置文件：`/etc/logrotate.conf`
```
  1 # see "man logrotate" for details
  2 # rotate log files weekly
  3 weekly
  4 
  5 # keep 4 weeks worth of backlogs
  6 rotate 4
  7 
  8 # create new (empty) log files after rotating old ones
  9 create
 10 
 11 # use date as a suffix of the rotated file
 12 dateext
 13 
 14 # uncomment this if you want your log files compressed
 15 #compress
 16 
 17 # RPM packages drop log rotation information into this directory
 18 include /etc/logrotate.d
 19 
 20 # no packages own wtmp and btmp -- we'll rotate them here
 21 /var/log/wtmp {
 22     monthly
 23     create 0664 root utmp
 24         minsize 1M
 25     rotate 1
 26 }
```

* 全局规则：
`weekly`     <==预设每个礼拜对日志档进行一次 rotate 的工作
 `rotate  4` <==保留几个日志文档呢？预设是保留四个！
 `create`    <== 回滚日志后，创建一个新的空文件来存储新的数据。
 
* 具体规则：
```
/var/log/wtmp {
    monthly
    create 0664 root utmp
        minsize 1M
    rotate 1
```
  说明：
  `/var/log/wtmp` { <==仅针对 /var/log/wtmp 所设定的参数
  `monthly` <==每个月一次，取代每周！
  `minsize 1M` <==档案容量一定要超过 1M 后才进行rotate (略过时间参数)
  `create 0664 root utmp` <==设定新建文件的权限 、所有者、用户组
  `rotate 1` <==仅保留一个，亦即仅有 wtmp.1 保留而已。 }


### 远程日志服务
配置远程日志服务器，实现日志集中管理：
* 配置SERVER端（接收端）
打开reception配置，日志为了准确，所以启动的是TCP链接。
```
[root@localhost ~]# vim /etc/rsyslog.conf
  
将：
  
# Provides UDP syslog reception
#$ModLoad imudp
#$UDPServerRun 514
  
# Provides TCP syslog reception
#$ModLoad imtcp
#$InputTCPServerRun 514
  
改为：
  
# Provides UDP syslog reception
#$ModLoad imudp
#$UDPServerRun 514
  
# Provides TCP syslog reception
$ModLoad imtcp
$InputTCPServerRun 514
  
  
#重启服务
[root@localhost ~]#systemctl restart rsyslog.service
  
#检查端口
[root@localhost ~]#netstat -anptu | grep 514
tcp        0     0 0.0.0.0:514       0.0.0.0:*               LISTEN      36217/rsyslogd
tcp6       0      0 :::514               :::*                  LISTEN      36217/rsyslogd
```
 

* 配置CLIENT端（发送端）
打开远程服务配置, 端口跟随服务器配置。
```bash
[root@xuegod69 ~]#vim  /etc/rsyslog.conf
  
将
  
# remote host is: name/ip:port, e.g. 192.168.0.1:514, port optional
#*.* @@remote-host:514
# ### end of the forwarding rule ###
  
改为：
  
# remote host is: name/ip:port, e.g. 192.168.0.1:514, port optional
*.* @@你自己远程服务器IP:514
# ### end of the forwarding rule ###
  
#重启
[root@xuegod69 ~]#systemctl restart rsyslog.service
```




### 扩展
#### wall命令介绍：
       wall -- send a message to everybody’sterminal.
```
[root@localhost ~]#wall Today is nice day!!!
wall Today is nicedayvim /etc/rsyslog.conf !
Broadcast message fromroot@localhost.localdomain (pts/0) (Thu Dec 17 22:10:28 2015):
Today is nice dayvim/etc/rsyslog.conf !
```

这样所有登录Linux的虚端的用户都会收到这个信息。

#### 如何防止日志被黑客删除呢
```
[root@localhost ~]#chattr +a /var/log/sshd.log
[root@localhost ~]#lsattr /var/log/sshd.log
-----a----------/var/log/sshd.log
```
加入了这个属性后，你的 /var/log/messages 登录档从此就仅能被增加，而不能被删除，直到 root 以『 chattr -a /var/log/messages 』取消这个 a 的参数后，才能被删除移！

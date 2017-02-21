title: 1-10 RHEL7 进程管理
date: 2015-12-7 15:29:44
tags: GOD linux
categories: linux
---
## 目录 
* [Linux进程管理](./#进程管理)
* [screen 实战](./#screen)

## 进程管理
程序与进程：
程序是静态的（文件），进程是动态的（运行的程序）。

进程和线程：
一个程序至少有一个进程,一个进程至少有一个线程. 
进程之间内存是独立
线程之前内存共享 ，高并发好一些 。 安全性差一些。

<!--more-->

### pstree
* 作用： 查看进程树
* 用法： pstree [-p] 
  显示进程号 -p
  
  ```
  pstree -p | more  # | more 是将结果分页显示，参考 more 命令
  ```

### tree 
* 作用： 显示目录树形结构
* 用法： tree 目录名

  ```
  ╰─$ tree /home/sam/back  | more
  /home/sam/back
  ├── 1_soft
  │ ├── QIpmsg_32_121019.tar.gz
  │ ├── vmware11
  │ │ ├── key
  │ │ └── VMware-Workstation-Full-11.0.0-2305329.x86_64.bundle
  │ └── xp.iso
  └── text
  ```

### ps
* 作用： 列出目前所有的正在内存当中的进程
* 用法： ps -aux # BSD的格式来显示进程 <br>       ps -ef # 是用标准的格式显示进程

```
$ ps -aux
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.0  36040  6028 ?        Ss   09:27   0:01 /usr/lib/systemd/systemd --swit
root         2  0.0  0.0      0     0 ?        S    09:27   0:00 [kthreadd]
root         3  0.0  0.0      0     0 ?        S    09:27   0:00 [ksoftirqd/0]
root         5  0.0  0.0      0     0 ?        S<   09:27   0:00 [kworker/0:0H]
root       434  0.0  0.3  74048 31784 ?        Ss   09:27   0:00 /usr/lib/systemd/systemd-journa
root       444  0.0  0.0  19656  1508 ?        Ss   09:27   0:00 /sbin/dmeventd
root       452  0.0  0.0  17596   168 ?        Ss   09:27   0:00 /sbin/lvmetad
```
  
  | 名称 | 含义 |
  |------|----------------------------------------------------------------------------------------------------|
  |USER|运行此进程的用户名|
  |PID|该 process 的号码。|
  |%CPU|该 process 使用掉的 CPU资源百分比|
  |%MEM|该 process 所占用的物理内存百分比|
  |VSZ|该 process 使用掉的虚拟内存量 (Kbytes)|
  |RSS|该 process 占用的固定的内存量 (Kbytes)|
  |TTY|该 process 是在那个终端机上面运作，若与终端机无关，则显示 ?，另外， tty1-tty6 是本机上面的登入者程序，若为 pts/0等等的，则表示为由网络连接进主机的程序。|
  |STAT|该程序目前的状态，Linux进程有5种基本状态： <br> `R` :(正在运行或在运行队列中等待)；<br> `S` :该程序目前正在睡眠当中 ，但可被某些讯号 (signal) 唤醒。<br> `T` :该程序目前暂停了  <br> `Z` ：该程序应该已经终止，但是其父程序却无法正常的终止他，造成 zombie (疆尸) 程序的状态 <br> `D` ：不可中断状态.  <br><br> `<` : 高优先级的 `N` : 低优先级的 `s` : 包含子进程 `l` : 多线程 `+` : 前台程序 |
  |START|该 process 被触发启动的时间|
  |TIME|该 process 实际使用 CPU运作的时间|
  |COMMAND|该程序的实际指令|
  
  注：
  * 正常先关闭子进程，再关闭父进程；一旦父进程先关闭而子进程没关闭，则子进程变成 zombie (疆尸) 状态
  * `ctrl-c` 是发送 SIGINT 信号，终止一个进程，`ctrl-z` 是发送 SIGSTOP信号，挂起一个进程

### free
* 作用 ：查看系统内存量
* 用法 : free ;  free [-m] #单位为M
`buffers` ： #缓存从磁盘读出的内容
`cached` ： #缓存需要写入磁盘的内容
```
╭─sam@sam  ~  
╰─$ free
             total       used       free     shared    buffers     cached
Mem:       8080868    5240964    2839904     375964       1144    2930824
-/+ buffers/cache:    2308996    5771872
Swap:      4193276          0    4193276
╭─sam@sam  ~  
╰─$ free -m
             total       used       free     shared    buffers     cached
Mem:          7891       5109       2782        367          1       2862
-/+ buffers/cache:       2245       5645
Swap:         4094          0       4094
```

### top
* 作用： 动态查看进程，统计信息区前五行是系统整体的统计信息。
* 用法： top ; top [-p] 进程号
默认3s刷新一次
`空格` ：立即刷新。
`q` ：退出
`M` ：按内存排序
`P` ： 按CPU排序
`<>` ： 翻页


```
╭─sam@sam  ~
╰─$ top
  
top - 15:44:29 up  6:17,  5 users,  load average: 0.28, 0.26, 0.31
Tasks: 271 total,   1 running, 270 sleeping,   0 stopped,   0 zombie
%Cpu(s):  1.0 us,  0.9 sy,  0.0 ni, 98.1 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem:   8080868 total,  5044804 used,  3036064 free,     1144 buffers
KiB Swap:  4193276 total,        0 used,  4193276 free.  2920332 cached Mem
  PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND                                                                      
 4590 sam       20   0 2918736 393244  56052 S 8.333 4.866  18:32.24 QQ.exe                                                                       
 2499 sam       20   0   18164   9620   1848 S 4.667 0.119  11:35.91 wineserver                                                                   
 2523 sam       20   0 1890980 447352 115104 S 1.667 5.536  20:57.55 firefox                                                                      
 8299 sam       20   0  662880  62120  48428 S 1.667 0.769   0:06.35 konsole                                                                      
 7989 sam       20   0 1354276 265240 169776 S 1.333 3.282   2:17.21 plugin-containe                                                              
 1054 root      20   0  854140 226432 197212 S 1.000 2.802   4:10.21 Xorg                                                                         
 2245 sam       20   0 3615704 218240 104364 S 1.000 2.701   4:18.24 plasma-desktop                                                               
 2341 sam        9 -11  516700  13032   9456 S 1.000 0.161   0:38.15 pulseaudio                                                                   
 2235 sam       20   0 3115760 111524  67328 S 0.333 1.380   2:11.33 kwin                                                                         
    1 root      20   0   36040   6028   3564 S 0.000 0.075   0:01.84 systemd                                                                      
    2 root      20   0       0      0      0 S 0.000 0.000   0:00.00 kthreadd                                                                     
    3 root      20   0       0      0      0 S 0.000 0.000   0:00.22 ksoftirqd/0         
```

#### 说明：
* 前五行，系统整体的统计信息

|行数|内容|含义|
|---------|-------------------------------|----------------------|
|第一行|top - 15:44:29 up  6:17,  5 users,  load average: 0.28, 0.26, 0.31| 第一行是任务队列信息 <br>  `15:44:29 up  6:17` : 当前时间  系统运行时间，格式为时:分 <br> `5 users,`: 当前登录用户数 <br> `load average: 0.28, 0.26, 0.31`: 系统负载，即任务队列的平均长度。 三个数值分别为  1分钟、5分钟、15分钟前到现在的平均值。
|第二行|Tasks: 271 total,   1 running, 270 sleeping,   0 stopped,   0 zombie|第二行为进程信息  <br>  `271 total` : 进程总数 <br> `1 running` :正在运行的进程数 <br> `270 sleeping` : 睡眠的进程数 <br> `0 stopped` : 停止的进程数 <br> `0 zombie` : 僵尸进程数  |
|第三行|%Cpu(s):  1.0 us,  0.9 sy,  0.0 ni, 98.1 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st|第三行为CPU的信息  <br>  `1.0% us` : 系统用户进程使用CPU百分比。 不包括调高优先级的进程。 CPU%是由每个核的CPU占用律之和算出来的。如果你是4核CPU，核1，CPU使用率为100%，核2，CPU使用率为100%。 则会出会CPU 高于100%的现象，最终为200% <br> `0.9% sy` : 内核中的进程占用CPU百分比 <br> `0.0% ni` : 用户进程空间内改变过优先级的进程占用CPU百分比 <br> `98.1% id` : 空闲CPU百分比|
|第四行|KiB Mem:   8080868 total,  5044804 used,  3036064 free,     1144 buffers|物理内存信息（KB） <br> `8080868 total` : 物理内存总量  <br> `5044804 used` : 使用的物理内存总量  <br> `3036064 free` : 空闲内存总量  <br> `2920332 buff/cache` : 用作内核缓存的内存量。|
|第五行|KiB Swap:  4193276 total,        0 used,  4193276 free.  2920332 cached Mem|交换区信息（KB） <br> `4193276 total` : 交换区总量  <br> `0k used` : 使用的交换区总量  <br> `4193276 free` : 空闲交换区总量  <br> `2920332 avail Mem` : 总的可利用内存是多少|

* 具体进程信息

|列名|含义|
|---------------|---------------------------------------------------|
|PID|进程id|
|USER|进程所有者的用户名|
|NI|进程优先级。 [nice](./#nice_renice)值。负值表示高优先级，正值表示低优先级|
|RES|实际使用内存大小。|
|S|进程状态。<br>  D=不可中断的睡眠状态<br>  R=运行<br>  S=睡眠<br>  T=跟踪/停止<br>  Z=僵尸进程|
|%CPU|上次更新到现在的CPU时间占用百分比|
|%MEM|进程使用的物理内存百分比|
|TIME+|进程使用的CPU时间总计，单位1/100秒|
|COMMAND|命令名/命令行|


### kill
* 作用 ： 控制（关闭）进程
* 用法 ： kill  给进程发送信号（停止进程） ； kill -9 pid； killall 服务名； pkill 服务名
常用信号：
`1`  HUP   重新加载配置文件。类似重启。
`2`  INT   和ctrl+c一样   一般用于通知前台进程组终止进程
`9`  KILL  强行中断
`19` STOP  和ctrl+z一样

### nice  renice
* 作用 ： nice 控制以什么优先级运行进程 。默认优先级是0；nice值 -20 ~ 19 越小优先级越高 普通用户0－19
renice #修改正在运行的进程的优先级
* 用法 ：  nice -n 优先级数字 进程号

### 前后台进程切换
#### jobs
列出所有后台进程

#### 创建后台指令
* nohup
* &
```
nohup <command> [argument…] &
```

#### fg
用法： fg 后期进程序列号
```
[root@localhost ~]# fg 1
```

## screen
虽然nohup很容易使用，但还是比较“简陋”的，对于简单的命令能够应付过来，对于复杂的需要人机交互的任务就麻烦了。
`screen` 可以恢复当时状态。

### 安装
rpm或yum方式

```
[root@localhost ~]# yum -y install screen
```

### 使用
* 启动screen
```
[root@localhost ~]# screen
```

* 执行业务操作
例如：
```
[root@localhost ~]# vim 1.sh
```

* 保存状态
按ctrl+a+d，出现[detached] 

* 恢复
查看 -ls ； 恢复 -r id号
```
╭─sam@sam  ~  
╰─$ screen -ls
There is a screen on:
        13033.pts-3.sam (Detached)
1 Socket in /run/uscreens/S-sam.
  
  
╭─sam@sam  ~  
╰─$ screen -r 13033
```






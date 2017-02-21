title: 1-4 rhel7文件管理和xfs文件系统备份
date: 2015-12-3 15:29:44
tags: GOD linux
categories: linux
---
## 目录
* Linux系统目录结构
* 相对/绝对路径
* 创建/复制/删除文件
* 查看文件内容
* xfs文件系统的备份和恢复

## 目录结构
详细的可以参考 [鸟哥](http://vbird.dic.ksu.edu.tw/linux_basic/0210filepermission_3.php)


| 目录 | 应放置文件内容 |
|-----|--------------------------------------------------------------------------------------------------|
|/    |通常称为根分区。所有的文件和目录皆由此开始。只有root用户对此目录拥有写权限。|
|/boot|存放Linux系统启动时需要加载的文件。 (一般在单独的一个磁盘分区里面保存) Kernel、grub等文件都存放在此。|
|/etc|RHEL6中主要存放服务的配置文件，RHEL7中，以/usr/lib/systemd/system进行代替|
|/var|是一个可增长的目录，包含很经常变的文件。例如，/var/log（系统日志）、/var/lib （包文件）|
|/root|管理员所有数据。  root用户的家目录|
|/tmp|临时文件存储位置|
|/usr|unix software source， 放置是所有系统默认的软件(distribution发布者提供的软件)，有点类似Windows 系统的『C:\Windows\ + C:\Program files\』这两个目录的综合体，鸟哥的链接有次级目录的说明|
|/opt|给第三方协力软件放置的目录，自行安装额外的软件(非原本的distribution提供的)|
|/bin|此目录包含二进制可执行文件，在单人维护模式下还能够被操作|
|/sbin|系统命令 ，此目录中的命令主要供系统管理员使用，为开机过程中所需要的，里面包括了开机、修复、还原系统所需要的指令，常见的指令包括：fdisk, fsck, ifconfig, init, mkfs等等。|
|/mnt |暂时挂载某些额外的装置|
|/media|放置的就是可移除的装置啦！ 包括软盘、光盘、DVD等等装置都暂时挂载于此。|
|/dev|包含设备文件。在Linux中，一切都被看做文件。终端设备、USB、磁盘等等都被看做文件|
|/home|普通用户所有数据存放在这个目录下|
|/proc|本身是一个『虚拟文件系统(virtual filesystem)』放置的数据都是在内存当中， 例如系统核心、行程信息(process)、周边装置的状态及网络状态等等。因为这个目录下的数据都是在内存当中， 所以本身不占任何硬盘空间啊！比较重要的文件例如：/proc/cpuinfo, /proc/dma, /proc/interrupts, /proc/ioports, /proc/net/* 等等。|
|/lib |系统最基本的动态链接共享库，尤其重要的是/lib/modules/这个目录， 因为该目录会放置核心相关的模块(驱动程序)|

<!-- more -->

## 绝对路径/相对路径
命令 cd
`.`  	表当前目录
`..`	代表的是上一级目录
` ` 表示当前目录

[](http://7xklqw.com1.z0.glb.clouddn.com/图片1.png)


## 创建/复制/删除文件
### touch　
作用：常用来创建空文件
语法： touch 文件名
```
[root@localhost ~]# touch sam
```

### mkdir
作用：创建目录
语法：mkdir 目录名
```
[root@localhost ~]# mkdir sam
[root@localhost ~]# mkdir -p sam/sam2/sam3
```
  `-p` 连同父目录一起创建

### Linux系统中一切皆文件
```
[root@localhost ~]# touch a
[root@localhost ~]# mkdir a
mkdir: cannot create directory ‘a’: File exists
```

### cp
作用：复制文件
语法：cp [-ra] 源文件  目标文件
-r :递归，包含子目录和文件
-a :复制属性
```
[root@localhost ~]# cp /etc/passwd  /root/
[root@localhost ~]# cp -r /boot/grub2/  sam/
```

### rm
作用：删除文件或目录
语法： rm [-rf]  文件或目录名
-r  递归删除（可以删除目录和目录里面的东西）
-f  强行删除
```
[root@xuegod163 ~]# rm -rf passwd
```

### mv
作用: 重命名
语法：mv 源：文件或目录名    目标：文件或目录名
```
[root@xuegod163 ~]# mv passwd  xiaochuan
```

### 经验：工作中，善用mv，慎用rm

## 查看文件内容
### cat
语法：cat 文件名
```
[root@localhost ~]# echo "I'm sam"  > sam 
[root@localhost ~]# cat sam 
I'm sam
```

### more　
语法： more　　查看文件名字
```
[root@localhost ~]# more  /etc/passwd
```
按下回车刷新一行，按下空格刷新一屏
`q`　退出


### less
语法： less　　查看文件名字
`q`　退出
使用光标键可以向上翻页

### linux中more与less的区别
more:不支持后退，但几乎不需要加参数，空格键是向下翻页，Enter键是向下翻一行，在不需要后退的情况下比较方便。
less：支持前后翻滚，既可以向上翻页（pageup按键），也可以向下翻页（pagedown按键）。空格键是向下翻页，Enter键是向下翻一行


### head
用法：head  -行号   文件名 (从第一行开始，查看文件，默认显示前10行)
或：  head   -n　数字（显示多少行） 文件名
```
[root@localhost ~]# head  -3 /etc/passwd 
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
```

### tail
从第后一行开始，查看文件，默认显示最后10行
用法：tail -n 数字 -f 文件名 或 tail -nf 文件名
-n  数字，显示多少行
-f  动态显示数据（不关闭）　　常用来查看日志
```
[root@localhost ~]# tail -12f  /var/log/alternatives.log
```

### 显示文件的第n行
```
[root@localhost ~]# head -n /etc/passwd | tail -1
[root@localhost ~]# tail -n /etc/passwd | head -1
```


## xfs文件系统的备份和恢复
XFS提供了 xfsdump 和 xfsrestore 工具协助备份XFS文件系统中的数据。xfsdump 按inode顺序备份一个XFS文件系统。与传统的UNIX文件系统不同，XFS不需要在dump前被卸载；对使用中的XFS文件系统做dump就可以保证镜像的一致性。这与XFS对快照的实现不同，XFS的dump和restore的过程是可以被中断然后继续的，无须冻结文件系统。xfsdump 甚至提供了高性能的多线程备份操作——它把一次dump拆分成多个数据流，每个数据流可以被发往不同的目的地。
### 测试准备
```
[root@localhost ~]# fdisk /dev/sda 
[root@localhost ~]# ls /dev/sda*
/dev/sda  /dev/sda1  /dev/sda2
[root@localhost ~]# partprobe /dev/sda #重新获取分区表
[root@localhost ~]# mkfs.xfs /dev/sda3 #格式化分区
[root@localhost ~]# mkdir /sda3 #创建挂载点
[root@localhost ~]# mount /dev/sda3  /sda3/ #挂载
```

### 备份
* 对整个分区进行备份
```
[root@localhost ~]# xfsdump -f /backup/dump_sda3  /sda3
......
please enter label for this dump session (timeout in 300 sec)
 -> dump_sda3   #指定备份标签
please enter label for media in drive 0 (timeout in 300 sec)
 -> media0   
#指定设备标签
```

  注意：`备份的设备这里不能写成/sda3/`

  非交互式进行备份
```
[root@localhost ~]# xfsdump -f /backup/dump_sda3  /sda3   -L dump_sda3  -M media0
```
  -L      指定备份标签
  -M	   指定设备标签

* 针对指定文件或目录进行备份
```
[root@localhost ~]# xfsdump -f /backup/dump_sda3_test -s test /sda3  -L dump_sda3_test -M media1
```
注意：`test sda3中间有空格，前后都不能加“/”`

### 查看备份信息
```
[root@localhost ~]# xfsdump -I
```

### 文件系统恢复
```
[root@localhost ~]# xfsrestore -f /backup/dump_sda3  /sda3/ #恢复整个分区
[root@localhost ~]# xfsrestore -f /backup/dump_sda3_test -s test /sda3 #只恢复单个的目录或文件
```
title: 1-14 Linux 文件系统
date: 2015-12-15 13:29:44
tags: GOD linux
categories: linux
---
## 目录
* 磁盘简介
* 文件系统架构
* 硬链接和软链接
* xfs和ext 文件系统
* 磁盘加密


## 磁盘简介
* windows CHS
![](http://7xklqw.com1.z0.glb.clouddn.com/cipan.png)
簇类似于Linux系统中的block
<!--more-->
  
* ZBR（Zoned Bit Recording）区位记录
Zoned-bit recording（ZBR 区位记录）是一种物理优化硬盘存储空间的方法，此方法通过将更多的扇区放到磁盘的外部磁道而获取更多存储空间。读外圈的数据快，读内圈的数据慢
ZBR磁盘扇区结构示意图
![](http://7xklqw.com1.z0.glb.clouddn.com/zbr.jpg)
  
操作系统读取硬盘的时候，不会一个个扇区地读取，这样效率太低，而是一次性连续读取多个扇区，即一次性读取一个"块"（block）。这种由多个扇区组成的"块"，是文件存取的最小单位。"块"的大小，最常见的是1KB，即连2个 sector组成一个 block。   4K =8扇区

* 查看系统块block大小

RHEL6操作系统：
```bash
[root@localhost~]# tune2fs -l /dev/sda1  | grep size
```

RHEL7操作系统
```bash
[root@localhost ~]#xfs_info /dev/sda1 | grep size
meta-data=/dev/sda1              isize=256    agcount=4, agsize=32000 blks
data     =                       bsize=4096   blocks=128000, imaxpct=25
```
注意： `block 提高磁盘读写性能， block值越大，读写速度越快，磁盘利用率越低；block值越小，读写速度越慢，磁盘利用率越高`

## 文件系统架构
Linux文件系统由三部分组成 ：文件名，inode，block（真正存数据）
`inode` ：文件数据都储存在"块"中，那么很显然，我们还必须找到一个地方储存文件的元信息，比如文件的创建者、文件的创建日期、文件的大小等等。这种储存文件元信息的区域就叫做inode，中文译名为"索引节点"。

### inode
#### inode的内容
inode包含文件的元信息，具体来说有以下内容：
* 文件的字节数
* 文件拥有者的User ID
* 文件的Group ID
* 文件的读、写、执行权限
* 文件的时间戳，共有三个：ctime指inode上一次变动的时间，mtime指文件内容上一次变动的时间，atime指文件上一次打开的时间。
* 链接数，即有多少文件名指向这个inode
* 文件数据block的位置

#### 查看inode信息
```bash
[root@localhost ~]#stat a.txt
  File: ‘a.txt’
  Size: 0           Blocks:0          IO Block: 4096   regular empty file
Device: fd01h/64769d  Inode: 34660829    Links: 1
Access:(0644/-rw-r--r--)  Uid: (    0/   root)   Gid: (    0/   root)
Access: 2015-12-1021:47:49.130938012 +0800
Modify: 2015-12-1021:47:49.130938012 +0800
Change: 2015-12-1021:47:49.130938012 +0800
```

#### inode的大小
inode也会消耗硬盘空间，所以硬盘格式化的时候，操作系统自动将硬盘分成两个区域。一个是数据区，存放文件数据；另一个是inode区（inode table），存放inode所包含的信息。

查看每个硬盘分区的inode总数和已经使用的数量，可以使用df命令。
```bash
[root@localhost ~]# df -i
Filesystem              Inodes  IUsed   IFree IUse% Mounted on
/dev/mapper/rhel-root 10485760131410 10354350    2% /
devtmpfs                501712    397  501315    1% /dev
tmpfs                   504204     6   504198    1% /dev/shm
tmpfs                   504204    511  503693    1% /run
tmpfs                   504204     13  504191    1% /sys/fs/cgroup
/dev/sr0                     0      0       0     - /mnt
/dev/sda1               512000    327  511673    1% /boot
```

#### 查看文件的inode号
使用ls-i命令，可以看到文件名对应的inode号码：
```
[root@localhost ~]# ls -i a.txt
34660829 a.txt
```

查看目录的inode号
```bash
╰─$ ll -di back 
951045 drwxr-xr-x 3 sam users 32 12月  1 15:20 back
```
 
## 硬链接和软链接

### 硬链接
语法 ： ln 源文件 目标文件
特点 ：
* 相链接文件的inode相同
* 硬链接不支持目录， 
* 硬链接不支持跨分区创建
* 源文件被删，不影响链接文件的正常使用

```bash
[root@localhost ~]# touch jjys
  
[root@localhost ~]# ln jjysdl.txt
  
[root@localhost ~]# ll -i  jjys dl.txt  #inode 相同
34660830 -rw-r--r-- 2 rootroot 10 Dec 10 22:06 dl.txt
34660830 -rw-r--r-- 2 rootroot 10 Dec 10 22:06 jjys
  
[root@localhost ~]# ln  wusheng/ fengchen
ln: ‘wusheng/’: hard link not allowed for directory #硬链接不支持目录
  
[root@localhost ~]# ln/boot/grub2/grub.cfg fengchen
ln: failed to create hard link‘fengchen’ => ‘/boot/grub2/grub.cfg’: Invalid cross-device link  #硬链接不支持跨分区创建
  
[root@localhost ~]# rm -rfjjys
[root@localhost ~]# cat dl.txt #源文件被删，不影响链接文件的正常使用
aaaa
bbbb
```

### 软链接：
语法 ： ln -s 源文件 目标文件
特点 ：相当于windows中的快捷方式
* 相链接文件的inode不相同
* 软链接支持目录， 
* 软链接支持跨分区创建
* 源文件被删，影响链接文件的正常使用

#### 跨分区创建软链接
```
[root@localhost ~]# ln  -s /boot/grub2/grub.cfg   fengye
```

#### 目录创建软链接
```
[root@localhost ~]# ln -s /boot/  yanmou
```

例：查看目录的链接数， `总数 -2 `
```
[root@localhost ~]# mkdir guoran
[root@localhost ~]# ll -d guoran/
drwxr-xr-x 2 root root 6 Dec 10 22:20 guoran/
  
[root@localhost ~]# ll -di guoran/
20039176 drwxr-xr-x 2 root root 6 Dec 10 22:20 guoran/
  
[root@localhost guoran]# ll -di .
20039176 drwxr-xr-x `2` rootroot 6 Dec 10 22:20 .
  
[root@localhost guoran]# mkdir zhengfa
[root@localhost guoran]# cd zhengfa/
[root@localhost zhengfa]# ll -di ..
20039176 drwxr-xr-x `3` rootroot 20 Dec 10 22:22 ..
```

#### 实例：inode数被用光
web服务器中小文件很多，导致硬盘有空间，但无法创建文件。
```
[root@localhost ~]# df -i
Filesystem              Inodes  IUsed   IFree IUse% Mounted on
/dev/mapper/rhel-root 10485760131421 10354339    2% /
devtmpfs                501712    397  501315    1% /dev
tmpfs                   504204      6  504198    1% /dev/shm
tmpfs                   504204    511  503693    1% /run
tmpfs                   504204     13  504191    1% /sys/fs/cgroup
/dev/sr0                     0      0       0     - /mnt
/dev/sda1               512000    327  511673    1% /boot
```

## xfs 和 ext 文件系统
xfs更适合大数据
* 数据完整性
采用XFS文件系统，当意想不到的宕机发生后，由于文件系统开启了日志功能，所以磁盘上的文件不再会意外宕机而遭到破坏，不论目前文件系统上存储的文件与数据有多少，文件系统都可以根据所记录的日志在很短的时间内迅速恢复磁盘文件内容
  
* 传输特性
xfs文件系统采用优化算法，日志记录对整体文件操作影响非常小。xfs查询与分配存储空间非常快。xfs文件系统能连续提供快速的反应时间。
  
* 可扩展性
xfs是一个全64-bit的文件系统，它可以支持上百万T字节的存储空间。对特大文件及小尺寸文件的支持都表现出众，支持特大数量的目录。最大可支持的文件大小为 9EB，最大文系统尺寸为18EB
  
* 传输带宽
XFS 能以接近裸设备I/O的性能存储数据。在单个文件系统的测试中，其吞吐量最高可达7GB每秒，对单个文件的读写操作，其吞吐量可达4GB每秒。

## 磁盘加密
LUKS(Linux Unified KeySetup)为Linux硬盘加密提供了一种标准。`cryptsetup` 这个工具提供实现。

### 硬盘分区
例如fdisk
```bash
[root@localhost ~]#fdisk /dev/sda
[root@localhost ~]#partprobe /dev/sda
```

### 安装加密工具：
```bash
[root@localhost ~]# rpm-qf `which cryptsetup`
cryptsetup-1.6.6-3.el7.x86_64
```
 

### 设置加密分区
设置密码
```bash
[root@localhost ~]# cryptsetup luksFormat /dev/sda3
WARNING!
=============
This will overwritedata on /dev/sda3 irrevocably.
Are you sure? (Typeuppercase yes): YES
Enter passphrase:  
Verify passphrase:  
```

### 映射
```bash
[root@localhost ~]# cryptsetup luksOpen  /dev/sda3  disk1
Enter passphrase for /dev/sda3:
[root@localhost ~]# ll /dev/mapper/disk1
lrwxrwxrwx 1 root root7 Dec 11 20:54 /dev/mapper/disk1 -> ../dm-2
```

### 格式化加密分区
```bash
[root@localhost ~]# mkfs.xfs  /dev/mapper/disk1
```

### 创建挂载点
```bash
[root@localhost ~]# mkdir /sda3
```

### 挂载
```bash
[root@localhost ~]# mount /dev/mapper/disk1  /sda3/
  
[root@localhost ~]# df -h
Filesystem             Size  Used Avail Use% Mounted on
/dev/mapper/rhel-root   10G 3.3G  6.8G  33% /
devtmpfs               2.0G     0 2.0G   0% /dev
tmpfs                  2.0G  140K 2.0G   1% /dev/shm
tmpfs                  2.0G  8.9M 2.0G   1% /run
tmpfs                  2.0G     0 2.0G   0% /sys/fs/cgroup
/dev/sr0               3.7G  3.7G    0 100% /mnt
/dev/sda1              497M  107M 391M  22% /boot
/dev/mapper/disk1     1019M  33M  987M   4% /sda3
```

### 关闭加密分区
```bash
[root@localhost ~]# umount /sda3/
  
[root@localhost ~]# cryptsetup luksClose /dev/mapper/disk1
  
[root@localhost ~]# mount /dev/sda3 /sda3/
mount: unknownfilesystem type 'crypto_LUKS'
```
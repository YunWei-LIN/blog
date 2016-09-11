title: 1-13 Linux 磁盘管理
date: 2015-12-14 14:29:44
tags: GOD linux
categories: linux
---
磁盘管理的一般步骤：
添加设备 --> 分区 --> 格式化（创建文件系统） --> 创建挂载点 --> 挂载 --> 修改配置文件

## 目录
* 分区
* 挂载
* 卸载
* 扩展swap

## 磁盘分区
按磁盘的大小，采用不同的方式。磁盘容量小于4T， 建议使用MBR分区；大于等于4T，采用GPT分区。

<!--more-->

### MBR  fdisk
MBR： Master boot record  ： 主引导记录
硬盘的0柱面、0磁头、1扇区称为主引导扇区（也叫主引导记录MBR）,总共512字节。
它由三个部分组成，主引导程序、硬盘分区表DPT（Disk Partition table）和分区有效标志（55AA）。
* 第一部分是主引导程序（boot loader）占446个字节，
* 第二部分是Partition table区（分区表），即DPT，占64个字节，16*4=64，硬盘中分区有多少以及每一分区的大小都记在其中。
* 第三部分是magic number，占2个字节，固定为55AA。
magic number：·结束标志字，偏移地址01FE--01FF的2个字节值为结束标志55AA,称为“魔数”（magic number）。如果该标志错误系统就不能启动。
linux中 使用 `fdisk` 工具进行管理。

<br> 
#### fdisk
```bash
[root@localhost ~]#fdisk -l                                  #查看设备使用状况
  
  
[root@localhost ~]#fdisk /dev/sda
Command (m for help): m
Command action
   a   toggle a bootable flag                                              #设置启动分区
   b  edit bsd disklabel
   c  toggle the dos compatibility flag
   d   delete a partition                                                     #删除分区
   g  create a new empty GPT partition table
   G  create an IRIX (SGI) partition table
   l  list known partition types
   m  print this menu                                                       #打印帮助页面
   n   adda new partition                                                  #创建一个新的分区
   o  create a new empty DOS partition table
   p  print the partition table                                           #打印分区表
   q   quit without saving changes                                   #退出不保存
   s  create a new empty Sun disklabel                     
   t  change a partition's system id
   u  change display/entry units
   v  verify the partition table
   w   write table to disk and exit                                      #保存
   x  extra functionality (experts only)
  
Command (m for help): n                                                  #创建一个新的分区
  
Partition type:
   p  primary (2 primary, 0 extended, 2 free)                           #创建一个主分区
   e  extended                                                                  #创建一个扩展分区
  
Select (default p): e
Partition number (3,4,default 3):                                        #指定分区号
First sector(26208256-41943039, default 26208256):            #指定开始柱面，默认回车就可以
Using default value26208256
Last sector, +sectorsor +size{K,M,G} (26208256-41943039, default 41943039): +1G         
#指定结束柱面，默认不指定的话是把剩下的所有空间分配给分区，等于说指定分区大小
  
Partition 3 of typeExtended and of size 5 GiB is set
  
Command (m for help): p                                                    #打印分区表
  
Disk /dev/sda: 21.5 GB,21474836480 bytes, 41943040 sectors
Units = sectors of 1 *512 = 512 bytes
Sector size(logical/physical): 512 bytes / 512 bytes
I/O size(minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier:0x000a705c
  
   Device Boot      Start         End      Blocks  Id  System
/dev/sda1   *       2048     1026047      512000  83  Linux
/dev/sda2         1026048    26208255   12591104   8e  Linux LVM
/dev/sda3        26208256    36694015    5242880    5  Extended      #表明是扩展 分区
  
Command (m for help): n
  
Partition type:
   p  primary (2 primary, 1 extended, 1 free)
   l  logical (numbered from 5)                                               #创建一个逻辑分区
Select (default p): l
Adding logicalpartition 5
First sector(26210304-36694015, default 26210304):
Using default value26210304
Last sector, +sectorsor +size{K,M,G} (26210304-36694015, default 36694015): +1G
Partition 5 of typeLinux and of size 1 GiB is set
  
Command (m for help): p
Disk /dev/sda: 21.5 GB,21474836480 bytes, 41943040 sectors
Units = sectors of 1 *512 = 512 bytes
Sector size(logical/physical): 512 bytes / 512 bytes
I/O size(minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier:0x000a705c
  
   Device Boot      Start         End      Blocks  Id  System
/dev/sda1   *       2048     1026047      512000  83  Linux
/dev/sda2         1026048    26208255   12591104   8e  Linux LVM
/dev/sda3        26208256    36694015    5242880    5  Extended
/dev/sda5        26210304    28307455    1048576   83  Linux
#逻辑分区的分区号从5开始，不管之前有没有第四或者第三个分区
  
Command (m for help): w                                            #保存
The partition table hasbeen altered!
Calling ioctl() tore-read partition table.
WARNING: Re-reading thepartition table failed with error 16: Device or resource busy.
The kernelstill uses the old table. The new table will be used at
the nextreboot or after you run partprobe(8) or kpartx(8)
Syncingdisks.
```

### GPT  parted
GPT，全局唯一标识分区表(GUID Partition Table)，GUID，与MBR最大4个分区表项的限制相比，GPT对分区数量没有限制，但Windows最大仅支持128个GPT分区。GPT可管理硬盘大小达到了18EB(1EB=1024PB=1,048,576TB)，不过NTFS格式最大仅支持256TB。

#### Parted 磁盘分区工具
```bash
[root@localhost ~]# rpm -qf `which parted`
parted-3.1-20.el7.x86_64
```

#### 查看parted命令的帮助信息：
```bash
[root@localhost ~]# parted --help
或
[root@localhost ~]# parted
GNU Parted 3.1
Using /dev/sda
Welcome to GNU Parted!Type 'help' to view a list of commands.
(parted) help            
(parted) quit                      #退出
```

#### 查看所有磁盘状态
```
[root@localhost ~]#parted -l
```
 

#### parted对磁盘进行操作
```
[root@localhost ~]#parted /dev/sdb
GNU Parted 3.1
Using /dev/sdb
Welcome to GNU Parted!Type 'help' to view a list of commands.
  
(parted) p                               #查看磁盘分区状态
Error: /dev/sdb:unrecognised disk label                    
  
(parted) mklabel                       #指定创建分区表类型为GPT
New disk label type? `gpt`
  
(parted) mkpart                  #创建分区                                              
Partition name?  []? `mydisk1`                                             
  
File system type?  [ext2]?       #指定分区文件系统类型，默认就可以，因为后期对分区进行格式化的时候，同样可以指定                                              
  
Start?  `1`                                  #指定分区起始位置                                                          
  
End? `100M`                        #指定分区结束位置                                                    
  
(parted) `p`                        #查看磁盘分区状态
Number  Start  End     Size    File system Name     Flags
 1     1049kB  99.6MB  98.6MB             mydisk1
  
(parted) `mkpart`                                                           
  
Partition name?  []? `2`                                                    
  
File system type?  [ext2]?                                               
  
Start? `100M`                                                               
  
End? `200M`                                                                 
  
(parted) `p`                                                               
  
Model: VMware, VMwareVirtual S (scsi)
Disk /dev/sdb: 21.5GB
Sector size(logical/physical): 512B/512B
Partition Table: gpt
Disk Flags:
  
Number  Start  End     Size    File system Name     Flags
 1     1049kB  99.6MB  98.6MB               mydisk1
 2     99.6MB  200MB   101MB                2
  
(parted) `quit`                                                             
  
Information: You mayneed to update /etc/fstab.
```


### 重新获取分区表
```
[root@localhost ~]#partprobe /dev/sda

[root@localhost ~]# ls /dev/sda*
/dev/sda  /dev/sda1 /dev/sda2  /dev/sda3  /dev/sda5
```

## 格式化
```
#mkfs.ext3 /dev/sdb1　　 或mkfs -t ext3 /dev/sdb1            #RHEL5 格式化
#mkfs.ext4 /dev/sdb1　　   或mkfs -t ext4 /dev/sdb1          #RHEL6 格式化
mkfs.xfs  /dev/sda5           或mkfs -t xfs /dev/sda5       #RHEL7 格式化
```


## 磁盘挂载
### 创建挂载点
```
  # 新建一个目录
[root@localhost ~]# mkdir /data
```

### 手动
* 挂载新磁盘
```
[root@localhost ~]# mount /dev/sda5 /data/
```

* 查看挂载状况
```
[root@localhost ~]# df -h
或
[root@localhost ~]#mount | grep data
```

### 开机自动挂载
开机时系统会自动加载 `/etc/fstab` 文件内容, 类似执行 `mount -a` 。我们为了开机自动挂载，也是修改此文件内容。
在文件最后新增一行，编辑需要新挂载的磁盘。
```
[root@localhost ~]# vim /etc/fstab
/dev/sda5             /sda5                    xfs          defaults         0                   0
```

各字段说明:

|/dev/sda5|/sda5|xfs|defaults|0|0|
|-----------|-------|---------|--------|---|---|
|`挂载的分区` | `挂载点` | 文件系统类型 | 挂载选项 | 是否备份 fs_dump |是否检测 fs_pass|

fs_dump： 是否要使用dump命令进行备份. 0为不备份，1为要备份。
fs_pass： 该字段被fsck命令用来决定在启动时是否需要被扫描的文件系统的顺序，根文件系统/对应该字段的值应该为1，其他文件系统应该为2。若该文件系统无需在启动 时扫描则设置该字段为0


#### UUID开机自动挂载
* UUID作用
UUID是一个标识你系统中的存储设备的字符串，其目的是帮助使用者唯一的确定系统中的所有存储设备，不管它们是什么类型的。它可以标识DVD驱动器，USB存储设备以及你系统中的硬盘设备等。

* 特点：
它是真正的唯一标志符。Linux中的许多关键功能现在开始依赖于UUID

* 获取设备的UUID
```
[root@localhost ~]# blkid | grep sda5
/dev/sda5:UUID="db81af84-5903-4907-b407-233716641819" TYPE="xfs"
```

* 自动挂载
```
[root@localhost ~]# vim /etc/fstab
UUID=db81af84-5903-4907-b407-233716641819       /sda5      xfs   defaults 0 0
```

#### 光盘开机自动挂载
```
[root@localhost ~]# vim /etc/fstab
/dev/sr0                /mnt                    iso9660   defaults       0 0
  # 或者：/dev/cdrom， /dev/sr0 = /dev/sr0
  
[root@localhost ~]# ll/dev/cdrom
lrwxrwxrwx 1 root root3 Dec 10  2015 /dev/cdrom -> sr0
```

### 特殊挂载方法：
```
 #指定格式 
[root@localhost ~]#mount -t xfs /dev/sda5  /sda5/
 #挂载ios光盘
[root@localhost ~]#mount -o loop rhel-server-7.1-x86_64-dvd   /media/
 # 
[root@localhost ~]#mount -o remount,ro  /sda5/
```

## 磁盘卸载
通过 `挂载点` 或 `挂载的分区` 来卸载磁盘
```
[root@localhost ~]#umount /dev/sda5
[root@localhost ~]#umount /data/
```

### 无法卸载
当其他用户正在使用磁盘时， 就会出现无法卸载； 可以通过 `lsof` 或 `fuser` 找到当前使用磁盘的进程，关闭此进程，再尝试卸载。
```
[root@localhost ~]#umount /dev/sda5
umount: /sda5: targetis busy.
        (In some cases useful info aboutprocesses that use
         the device is found by lsof(8) or fuser(1))
  
[root@localhost ~]#lsof /sda5/
COMMAND  PID USER  FD   TYPE DEVICE SIZE/OFF NODENAME
bash    3967 root cwd    DIR    8,5       6  128 /sda5
  
[root@localhost ~]#kill -9 3967
  
[root@localhost ~]#fuser -m -u -v /sda5/
                     USER        PID ACCESS COMMAND
/sda5:               root     kernel mount (root)/sda5
                     root      36749 ..c.. (root)bash
  
[root@localhost ~]#kill -9 36749
  
[root@localhost ~]#umount /dev/sda5
```

## 扩展swap
当swap分区空间不够时， 可以通过以下方式扩展swap大小。

### 新建swap 分区
```
 #格式化sdb1分区为swap格式
[root@localhost ~]# mkswap /dev/sdb1
Setting up swapspaceversion 1, size = 96252 KiB
no label,UUID=d6e96d3b-9d74-4eda-8bdf-387754dca90e
```

### 查看虚拟内存大小
```
[root@localhost ~]#free -m
              total        used        free      shared buff/cache   available
Mem:           3939         517        2942           9         479        3191
Swap:          2047           0        2047
```

### 启动swap分区
```
[root@localhost ~]#swapon /dev/sdb1
```

### 再次查看
```
[root@localhost ~]#free -m
              total       used        free      shared buff/cache   available
Mem:           3939         516        2942           9         479        3191
Swap:          2141           0        2141
```
 

### 实现开机自动挂载swap分区
```
[root@localhost~]# vim /etc/fstab
/dev/sdb1               swap                    swap    defaults        0 0
```
说明：只有重启才能生效，mount –a 无法自动扩展swap分区的。

### 关闭
```
[root@localhost ~]#swapoff /dev/sdb1
```
---
title: U盘引导多个linux镜像
date: 2017-06-27 19:08:03
tags: [linux, usb, grub, BIOS, UEFI]
categories: linux
---

___主要内容___

一个U盘在手，走遍天下都不怕。
关键就是这个U盘，
它引导多个linux镜像安装,同时支持BIOS和UEFI模式 。不需要安装系统时就是一个普通U 盘。

话说我一个开发，怎么会研究这个玩意。


<!-- more -->

## 前期准备
+ U盘
最好16G以上，不然装不了几个系统的。

+ 预备知识
在此特别感谢[Feng_Yu](https://my.oschina.net/abcfy2/blog/491140?nocache=1497879527986) 的帮助， 把他的内容读通，基本步骤就参照他的。
最大的区别是我把U盘分了多个区，除了`GPT分区(EF00)`和`bios_grub分区(EF02)`，其他的分区平常都可以当普通U盘使用。

## 格式化U盘
我在 `Opensuse 42.1` 系统操作的。

### U盘格式化为GPT分区
工具: `parted`, `gdisk`。
#### 分区
使用图像化工具 `parted` 将U盘格式化并建立分区。
+ 分区一： GPT分区，256Mb， 安装 `Grub`
+ 分区二： ext4分区，5G，数据盘
+ 分区三： NTFS分区，剩下空间-1M，数据盘
+ 分区四： 剩余最后1M，不要格式化！！！！！！！
分区二和分区三平常都可当一般U盘使用， 分区三 为兼容 `windows`系统， 分区二为安装 `centos` 等不支持 `ntfs`的系统。

#### bios_grub
使用`gdisk` 将最后一个分区标记为 `EF02`， 即`bios_grub分区`
```sh
╰─# gdisk /dev/sdb
GPT fdisk (gdisk) version 0.8.8

Partition table scan:
  MBR: protective
  BSD: not present
  APM: not present
  GPT: present

Found valid GPT with protective MBR; using GPT.

Command (? for help): n
Partition number (3-128, default 3): 
First sector (34-31266782, default = 31264768) or {+-}size{KMGTP}: 
Last sector (31264768-31266782, default = 31266782) or {+-}size{KMGTP}: 
Current type is 'Linux filesystem'
Hex code or GUID (L to show codes, Enter = 8300): EF02
Changed type of partition to 'BIOS boot partition'

Command (? for help): p
Disk /dev/sdb: 31266816 sectors, 14.9 GiB
Logical sector size: 512 bytes
Disk identifier (GUID): 42BF9B08-57CA-4F18-8CB5-C3B06668B141
Partition table holds up to 128 entries
First usable sector is 34, last usable sector is 31266782
Partitions will be aligned on 2048-sector boundaries
Total free space is 2014 sectors (1007.0 KiB)

Number  Start (sector)    End (sector)  Size       Code  Name
   1            2048          526335   256.0 MiB   EF00  
   2          526336        31264767   14.7 GiB    0700  
   3        31264768        31266782   1007.5 KiB  EF02  BIOS boot partition

Command (? for help): w

Final checks complete. About to write GPT data. THIS WILL OVERWRITE EXISTING
PARTITIONS!!

Do you want to proceed? (Y/N): y
OK; writing new GUID partition table (GPT) to /dev/sdb.
The operation has completed successfully.
```

## 安装grub到U盘
```sh
$ sudo mount /dev/sdb1 /mnt -o uid=$USER,gid=$USER  # 没什么好说的，挂载U盘使用，加上uid和gid参数只是为了编辑文件不需要sudo而已

# grub安装到MBR
$  sudo grub2-install --target=i386-pc --recheck --boot-directory=/mnt/boot /dev/sdb
Installing for i386-pc platform.
Installation finished. No error reported.

# grub安装到ESP,特别注意--removable参数，安装到移动设备上一定要用这个参数
$ sudo grub2-install --target x86_64-efi --efi-directory /mnt --boot-directory=/mnt/boot --removable
Installing for x86_64-efi platform.
Installation finished. No error reported.
```
`grub2-install` 命令根据你的系统决定， 也可能是 `grub-install`

## 添加grub菜单
在`/mnt/boot/grub2`这个目录下，新建`grub.cfg`配置文件
例如： 

```sh
# path to the partition holding ISO images (using UUID)
probe -u $root --set=rootuuid
set imgdevpath="/dev/disk/by-uuid/$rootuuid"

# lsblk -f to find your UUID for data device
set ntfs_data_uuid="71E161DA3EB70972"
set ext4_data_uuid="50a92fdb-4e11-47b8-889c-02f430d3a780"

# define globally (i.e outside any menuentry)
insmod search_fs_uuid
search --no-floppy --set=isopart --fs-uuid $ntfs_data_uuid
search --no-floppy --set=centos_iso_part --fs-uuid $ext4_data_uuid   # centos no ntfs
set isopath=/iso  # 定义iso文件存放的路径

# 定义各种要启动的iso镜像名
set ubuntu=ubuntu-16.04.1-desktop-amd64.iso                  # ubuntu桌面版镜像,支持12.04-16.04
set ubuntuserver=ubuntu-16.04.1-server-amd64.iso             # ubuntu服务器版镜像,支持14.04-16.04
set debianlive=debian-live-8.6.0-amd64-gnome-desktop.iso     # debian livecd镜像,支持debian8
set deepin15=deepin-15.3-amd64.iso                           # deepin15镜像
set linuxmint=linuxmint-18.1-kde-64bit.iso                   # linuxmint镜像,支持17-18
set ubuntukylin=ubuntukylin-16.04.1-desktop-amd64.iso        # ubuntukylin镜像,支持14.04-16.04
set centosdvd=CentOS-7-x86_64-DVD-1511.iso                   # centos DVD版镜像,包括DVD/Everything/Minimal/Netinstall镜像,支持6-7
set centoslive=CentOS-7-x86_64-LiveGNOME-1511.iso            # centos livecd镜像,包括livecd/livedvd，支持7(6似乎不支持直接引导ISO,期待PR解决)
set opensuse=openSUSE-Leap-42.1-DVD-x86_64.iso               # opensuse DVD安装镜像


# grub模块配置
loadfont unicode
set lang=zh_CN
insmod all_video
insmod gfxterm
insmod gettext
insmod gfxmenu
insmod png
insmod ext2
insmod part_msdos
insmod xfs
insmod ntfs
insmod loopback
insmod iso9660
terminal_output gfxterm
set menu_color_normal=cyan/blue
set menu_color_highlight=white/blue
gfxmode keep
set vt_handoff=vt.handoff=7

# 主题配置
set gfxmode=auto
set timeout_style=menu
set timeout=10
set theme=$prefix/themes/Tuxkiller2/theme.txt
export theme

menuentry '从本地硬盘启动' --class harddrive {
  set root=(hd1)
  chainloader +1
}

if [ -f "($isopart)$isopath/$linuxmint" ]; then
  menuentry "linuxmint-18.1-kde-64" --class linuxmint --class gnu-linux --class gnu --class os {
    set isofile="$isopath/$linuxmint"
    loopback loop ($isopart)$isofile
    echo '载入Linux Mint ...'
    linux (loop)/casper/vmlinuz file=/cdrom/preseed/linuxmint.seed boot=casper iso-scan/filename=$isofile noeject noprompt splash locale=zh_CN.UTF-8 --
    echo '载入初始化内存盘 ...'
    initrd (loop)/casper/initrd.lz
  }
fi


if [ -f "($isopart)$isopath/$ubuntu" ]; then
  menuentry "Ubuntu Desktop ISO" --class ubuntu --class gnu-linux --class gnu --class os {
    set isofile="$isopath/$ubuntu"
    loopback loop ($isopart)$isofile
    echo '载入Ubuntu Desktop ...'
    linux (loop)/casper/vmlinuz.efi file=/cdrom/preseed/ubuntu.seed boot=casper iso-scan/filename=$isofile noeject noprompt splash locale=zh_CN.UTF-8 --
    echo '载入初始化内存盘 ...'
    initrd (loop)/casper/initrd.lz
  }
fi

if [ -f "($isopart)$isopath/$ubuntukylin" ]; then
  menuentry "UbuntuKylin Desktop ISO" --class ubuntukylin --class gnu-linux --class gnu --class os{
    set isofile="$isopath/$ubuntukylin"
    loopback loop ($isopart)$isofile
    echo '载入UbuntuKylin Desktop ...'
    linux (loop)/casper/vmlinuz.efi boot=casper iso-scan/filename=$isofile noeject noprompt splash locale=zh_CN.UTF-8 --
    echo '载入初始化内存盘 ...'
    initrd (loop)/casper/initrd.lz
  }
fi

# 特别注意: Ubuntu Server会出现安装的时候检测不到光驱的现象
# 此时手工进入shell下，将iso镜像挂载在/cdrom继续即可
# mount /dev/sdb1 /media
# mount -o loop /media/boot/iso/ubuntu-*-server-*.iso /cdrom
if [ -f "($isopart)$isopath/$ubuntuserver" ]; then
  menuentry "Ubuntu Server ISO" --class ubuntu --class gnu-linux --class gnu --class os{
    set isofile="$isopath/$ubuntuserver"
    loopback loop ($isopart)$isofile
    set gfxpayload=keep
    echo '载入Ubuntu Server ...'
    linux (loop)/install/vmlinuz file=/cdrom/preseed/ubuntu-server.seed iso-scan/filename=$isofile quiet ---
    echo '载入初始化内存盘 ...'
    initrd (loop)/install/initrd.gz
  }
fi

if [ -f "($isopart)$isopath/$deepin15" ]; then
  menuentry "Deepin 15 ISO" --class deepin --class gnu-linux --class gnu --class os{
    set isofile="$isopath/$deepin15" 
    loopback loop ($isopart)$isofile
    echo '载入Deepin ...'
    linux (loop)/live/vmlinuz.efi boot=live config findiso=$isofile noeject noprompt locales=zh_CN.UTF-8 --
    echo '载入初始化内存盘 ...'
    initrd (loop)/live/initrd.lz
  }
fi

if [ -f "($isopart)$isopath/$debianlive" ]; then
  menuentry "Debian LiveCD ISO" --class debian --class gnu-linux --class gnu --class os{
    set isofile="$isopath/$debianlive"
    loopback loop ($isopart)$isofile
    echo '载入Debian LiveCD ...'
    linux (loop)/live/vmlinuz boot=live config findiso=$isofile noeject noprompt locales=zh_CN.UTF-8 --
    echo '载入初始化内存盘 ...'
    initrd (loop)/live/initrd.img
  }
fi

if [ -f "($centos_iso_part)$isopath/$centosdvd" ]; then
  menuentry "CentOS-7-x86_64-DVD-1511" --class centos --class gnu-linux --class gnu --class os{
    set isofile="$isopath/$centosdvd"
    loopback loop ($centos_iso_part)$isofile
    echo '载入CentOS DVD ...'
    linux (loop)/isolinux/vmlinuz inst.stage2=hd:UUID=$ext4_data_uuid:/$isofile inst.lang=zh_CN.UTF-8 rhgb
    echo '载入初始化内存盘 ...'
    initrd (loop)/isolinux/initrd.img
  }
fi


if [ -f "($isopart)$isopath/$opensuse" ]; then
menuentry 'openSUSE-42.1-DVD-x86_64' --class opensuse --class gnu-linux --class gnu --class os{
    set isofile="$isopath/$opensuse"
    loopback loop ($isopart)$isofile
    linux (loop)/boot/x86_64/loader/linux install=hd:$isofile
    initrd (loop)/boot/x86_64/loader/initrd
}
fi

if [ -f "($isopart)$isopath/$centoslive" ]; then
  menuentry "CentOS LiveCD ISO" --class centos --class gnu-linux --class gnu --class os{
    set isofile="$isopath/$centoslive"
    loopback loop ($isopart)$isofile
    echo '载入CentOS LiveCD ...'
    probe -l -s isolable (loop)
    linux (loop)/isolinux/vmlinuz0 boot=isolinux root=live:LABEL="$isolable" iso-scan/filename=$isofile rd.live.image LANG=zh_CN rd.locale.LANG=zh_CN rhgb
    echo '载入初始化内存盘 ...'
    initrd (loop)/isolinux/initrd0.img
  }
fi

menuentry '关闭系统' --class halt {
  echo '关机中 ...'
  halt
}

menuentry '重启系统' --class reboot {
  echo '重启系统中 ...'
  reboot
}
```


## 美化
 美化启动
```sh
# 对应的把下面的/dev/sdb1替换成你的U盘的ESP分区
sudo mount /dev/sdb1 /mnt/  -o uid=$USER,gid=$USER,utf8=1
cp -R themes/ /mnt/boot/grub2/
```

`themes` 请参考 [Feng_Yu的](http://git.oschina.net/abcfy2/grub-cfg)
 

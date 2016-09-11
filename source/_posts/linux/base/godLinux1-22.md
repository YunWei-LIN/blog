title: 1-22 系统救援
date: 2015-12-24 12:29:44
tags: GOD linux
categories: linux
---
## 主要内容
* 光盘启动，系统救援
* 忘记root用户的密码
* 磁盘资源耗尽故障
* 双系统启动修复

<!-- more -->

## 系统救援
当系统坏了，进不去了。  进入救援模式拷贝数据
以光盘引导：
![](http://7xklqw.com1.z0.glb.clouddn.com/rescue1.png)
![](http://7xklqw.com1.z0.glb.clouddn.com/rescue2.png)
![](http://7xklqw.com1.z0.glb.clouddn.com/rescue4.png)

`注意：需要切换文件系统根 chroot  /mnt/sysimage  `
![](http://7xklqw.com1.z0.glb.clouddn.com/rescue5.png)
![](http://7xklqw.com1.z0.glb.clouddn.com/rescue6.png)
![](http://7xklqw.com1.z0.glb.clouddn.com/rescue8.png)

此时可以copy各种需要的文件。

## 忘记root用户的密码
解决方法：重启系统进入单用户模式，然后重设密码
把下图中的x去掉，然后reboot 就可以，再次使用root进入密码，就不需要输入密码。
![](http://7xklqw.com1.z0.glb.clouddn.com/root_passwd.png)
进入系统passwd就可以修改root。

## 磁盘资源耗尽故障
现象：
    
    无法写入新的文件，提示“… :设备上没有空间”
    部分程序无法运行，甚至系统无法启动


故障原因:
    
    磁盘空间已被大量的数据占满，空间耗尽
    虽然还有可用空间，但文件数i节点耗尽
      
解决方案：

    清理磁盘空间，删除无用、冗余的文件
    转移或删除占用大量i节点的琐碎文件
    进入单用户模式、救援模式进行修复或删除文件

## 双系统启动修复
当我们安装双系统环境，先安装Linux再安装Windows；；或者已经安装好双系统环境的Windows损坏，在重新安装Windows后，保存 GRUB的MBR（MasterBoot Record，主引导记录）会被Windows系统的自举程序NTLDR所覆盖，造成Linux系统无法引导。

恢复步骤：
* 如果要恢复双系统引导，首先用上述方法进入救援模式，执行chroot命令如下：
```
    sh-4.1#chroot /mnt/sysimage
```

* 将根目录切换到硬盘系统的根目录中，然后执行grub-install命令重新安装GRUB：
```
    sh-4.1#grub-install  /dev/sda
```
![](http://7xklqw.com1.z0.glb.clouddn.com/grub-install.png)

* 然后依次执行exit命令，退出chroot模式及救援模式（执行两次exit命令）：
```
    sh-4.1#exit
    sh-4.1#reboot
```
系统重启后，将恢复GRUB引导的双系统启动









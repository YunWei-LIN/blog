title: 1-1 测试环境- rhel7 安装
date: 2015-11-24 15:29:44
tags: GOD linux
categories: linux
---
## 虚拟机安装
vmware workstation 10 以上
创建空的虚拟机，一路next，网络建议选择桥接， 注意选择好系统类型。
<!--more-->
## rhel7 安装
* 加载rhel7系统镜像, 启动虚拟机
![](http://7xklqw.com1.z0.glb.clouddn.com/2015-11-24_1.jpg)

* 启动成功，开始安装
![](http://7xklqw.com1.z0.glb.clouddn.com/rhel.png)
第一项正常安装。回车， 开始安装。
需要时可以选择第二项，检查镜像完整性；第三项可以用来恢复系统。

* 选择系统语言
中文在最下面， 正式环境建议选择英文
![](http://7xklqw.com1.z0.glb.clouddn.com/2015-11-24_2.jpg)

* 详细配置
rhel7的详细配置都在一个界面上， 需要把所有 `！` 图案都解决才能继续
![](http://7xklqw.com1.z0.glb.clouddn.com/2015-11-24_3.jpg)

  1. software selection
  根据自己需要选择，新手建议 最后的 `server with GUI`
  ![](http://7xklqw.com1.z0.glb.clouddn.com/2015-11-24_5.jpg)

  2. kdump
  异常日志，可以把这个关闭
  ![](http://7xklqw.com1.z0.glb.clouddn.com/2015-11-24_6.jpg)
  
  3. INSTALLATION DESTINATION 分区
  可以根据需要手动分区。 虚拟机安装时 `+` 可能会出不来， `Done` 按钮确认， 重新分区，多刷几次应该能把 `+` 显示出来。
  建议`/home` 分区 和 系统分区分开。
  分区大小：
  `/boot`： 256M 或 512M
  `swap` ： 物理内存的1.5～2倍， 最大4G， 再大完全没意义。
  ![](http://7xklqw.com1.z0.glb.clouddn.com/2015-11-24_7.jpg)
  ![](http://7xklqw.com1.z0.glb.clouddn.com/2015-11-24_8.jpg)
  ![](http://7xklqw.com1.z0.glb.clouddn.com/2015-11-24_9.jpg)
  
  接受 改变：
  ![](http://7xklqw.com1.z0.glb.clouddn.com/2015-11-24_10.jpg)
  
  所有 `！` 图案都解决， 点击 `Begin installtion` ， 开始安装
  ![](http://7xklqw.com1.z0.glb.clouddn.com/2015-11-24_11.jpg)
  
* 管理员密码和一般用户
安装过程中，可以设置 管理员root密码和建立一个一般用户
![](http://7xklqw.com1.z0.glb.clouddn.com/2015-11-24_14.jpg)
![](http://7xklqw.com1.z0.glb.clouddn.com/2015-11-24_12.jpg)
![](http://7xklqw.com1.z0.glb.clouddn.com/2015-11-24_13.jpg)

* 安装完成
安装完成，系统会自动重启，用刚刚设置的账户密码进入，第一次会要求激活
![](http://7xklqw.com1.z0.glb.clouddn.com/2015-11-24_15.jpg)

* 启动网络
第一次似乎网络似乎是关闭的， 点击右上角 小电脑 图案，启用网络。
![](http://7xklqw.com1.z0.glb.clouddn.com/2015-11-24_16.jpg)

## 虚拟机快照
为防止测试破坏， 可以建立虚拟机快照，可在虚拟机关闭的情况下做快照，一个快照大概2G大小。一旦虚拟机被破坏，可以快速恢复。
![](http://7xklqw.com1.z0.glb.clouddn.com/2015-11-24_17.jpg)


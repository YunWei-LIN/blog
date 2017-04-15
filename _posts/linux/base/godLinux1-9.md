title: 1-9 文件归档和解压缩
date: 2015-12-6 20:29:44
tags: GOD linux
categories: linux
---
## 文件归档和解压缩
### file
* 作用：file 确定 filetype
* 用法：file /etc/passwd
  注：`linux系统不根据后缀名识别文件类型，用file命令查看文件的类型。`

```
[sam1@localhost xuegod]$ file a
a: directory
[sam1@localhost xuegod]$ file p.txt 
p.txt: ASCII text
[sam1@localhost xuegod]$ 
```

### tar命令
* 作用：打包、压缩文件
* 用法：tar [z/j][c/x][vf] archive文件名  源文件（目录）
    压缩方式： z=gz压缩；  j=bz2压缩
    类型：    c=归档（压缩）；    x：解压
    v：详细
    f：filename
```
[root@localhost ~]# tar -zcvf grub1.tar.gz /boot/grub2/
[root@localhost ~]# tar -jcvf grub2.tar.bz2 /boot/grub2/
```



### zip
* 作用：zip是压缩程序，unzip是解压程序。
* 用法：zip [-r] 文件源（/目录）
  unzip archive文件名
```
[root@localhost ~]# zippasswd.zip /etc/passwd #压缩文件：
[root@localhost ~]# zip-r grub2.zip /boot/grub2/ # -r 压缩目录
```
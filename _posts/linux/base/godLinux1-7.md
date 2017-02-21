title: 1-7 文件权限管理
date: 2015-12-5 18:29:44
tags: GOD linux
categories: linux
---
## 主要内容
* 文件基本权限
* 文件高级权限

## 文件基本权限
### 查看文件权限
```bash
$ ll $filename
```

### 基本权限构成
```
$ ll text 
-rw-r--r-- 1 sam users 0 12月  1 15:20 text
```
| -  | rw- |　r--　　　| 　r--　　　| 　sam　| users　|　text(FILENAME) |
|----|----|----|----|----|----|----|
| 类型 | 拥有者的权限 | 所属组的权限 | 其他人的权限 | 拥有者 | 属组 | 对象 |

<!--more-->

对于文件：r读　　w写　　x执行
对于目录：r读（看到目录里面有什么）       cat   more  less   ls
　　　　　w建文件、删除、移动                  touch   mkdir   rm   mv  cp
　　　　　x进入                                        cd

### 修改权限
#### chmod
作用：修改文件权限
u-w               user                      拥有者
g+x               group                   组
o=r                other                    其他人
a+x               all                        所有人
`-` 代表移除； `+` 代表增加； `=` 代表替换
此命令同样可以修改目录， 用法和修改文件一致

* 数字表示权限 

| r-- | -w- | --x | 进制 |
|-----|------|-----| ----|
|100  |  010 | 001 |二进制 |
|4    |2     |1    |十进制 |

用数字表示就是把对应数据相加
so `rwx` 表示为 7； `rw` 为 6;
rwxr-xr-x 的值是多少？rwx=4+2+1=7;r-x=4+1=5;r-x=4+1=5; rwxr-xr-x=755 

#### chown
作用：修改文件拥有者和所属组
语法：chown USER:GROUP 对象
chown USER 对象
chown :GROUP 对象 `:` 必须
-R 递归(目录下的所有内容全部更改，否则只修改目录)
一个文件只有读的权限，拥有者是否可以写这个文件？   文件所有者一定可以写文件

### 默认权限
* 计算方法：
** 快速计算
文件默认权限＝666（默认值）-umask值
目录默认权限＝777（默认值）-umask值
这是一个快速计算方法，但不严谨。
** 完整算法
在2进制表示的情况下
默认值 && （umask值取反）
** umask值
用户默认的shell配置文件中设置，如 `/etc/bashrc`
```
if [ $UID -gt 199 ]&& [ "`id -gn`" = "`id -un`" ]; then  # id -gn显示组名，id -un 显示用户名
        umask 002   ＃普通用户
else
        umask 022   ＃系统用户
fi
```


## 高级权限
### 特殊权限
SUID               SGID                 Stickybit
s对应的数值为：u 4，g  2，o   1

* SUID：
限定：只能设置在二进制可执行程序上面。对目录文本设置无效。
功能：程序运行时的权限从执行者变更成程序所有者。
可用 `chmod` 修改
```
[root@localhost ~]#chmod u+s /usr/bin/less
[root@localhost ~]# ll /usr/bin/less
-rwsr-xr-x. 1 root root158240 Feb  4  2014 /usr/bin/less
```
注意：
`[root@localhost ~]#chmod u+s /usr/bin/less` 等同于 `[root@localhost ~]#chmod 4755 /usr/bin/less`

* SGID：
限定：既可以给二进制可执行程序设置，也可以给目录设置。
功能：在设置了SGID权限的目录下建立文件时，新创建的文件的所属组会继承上级目录的所属组
```
[root@localhost ~]#chmod g+s test/
```

* Stickybit
限定：只作用于目录
功能：目录下创建的文件只有root、文件创建者、目录所有者才能删除
```
[root@localhost ~]#chmod o+t test
```

### 扩展ACL
* 查看 getfacl
```
[root@localhost ~]#getfacl test
# file: test
# owner: root
# group: root
user::rw-
group::r--
other::r--
```

* 设置 setfacl `-m` 必须
```
[root@localhost ~]#setfacl -m u:sam1:rwx test
```

* 对目录进行设置
```
[root@localhost ~]#setfacl -R -m u:sam1:rw- test
```
  (`-R` 一定要在 `-m` 前面，表示目录下所有文件）

* 删除acl
```
[root@localhost ~]#setfacl -x u:sam1 test #删除单个用户的权限
[root@xuegod163~]# setfacl -b test         #删除所有acl权限
```

### 文件系统扩展属性
* 命令
`chattr`：设置， `lsattr`：查看
```
chattr [+/-] [a/i] $filename
```
  `+` 增加属性， `-` 减少属性； `a` ：只能追加内容， `i`：不能被修改
  如 `+a`= 只能追加内容， `+i`= 不能被修改
```
[root@localhost ~]#chattr +i test   #root 都不能删除
```
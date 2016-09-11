title: 1-11~12 - 重定向和文件查找
date: 2015-12-9 15:28:44
tags: GOD linux
categories: linux
---
## 目录
* 重定向
* 管道｜
* 文件查找

## 重定向
### 文件描述符
文件描述符:内核（kernel）利用文件描述符（file descriptor）来访问文件。文件描述符是非负整数。打开现存文件或新建文件时，内核会返回一个文件描述符。读写文件也需要使用文件描述符来指定待读写的文件。

linux下一切皆文件。
STDIN  标准输入   文件描述符为：0  如：键盘文件
STDOUT 标准输出   文件描述符为：1 如：屏幕终端
STDERR 错误输出   文件描述符为：2 如：屏幕终端

<!-- more -->

### 输出重定向
作用： `>` `>>` 代表输出重定向；
  `>` 内容覆盖。
  `>>` 内容末尾追加，
用法： [文件描述符][`>` / `>>`] 目的文件


#### 标准输出 
`1>` 表示stdout标准输出，系统默认值是1， 所以 `1>` 等价 `>`
```
[root@localhost ~]#echo 123456 | passwd --stdin sam > /dev/null  #/dev/null 代表空设备文件，就是Linux中的黑洞。放多少东西都填不满。
[root@localhost ~]#echo 123456 | passwd --stdin sam 1> /dev/null
  
[root@localhost ~]# ls/home/  1>  a.txt
[root@localhost ~]# cat a.txt
sam
```

#### 标准错误
`2>` 表示STDERR 错误输出
```
[root@localhost ~]# ls/homee  2>  b.txt
  
[root@localhost ~]# cat b.txt
ls: cannot access/homee: No such file or directory
```

#### 混合输出 
`&>` 表示混合输出 
```
[root@localhost ~]# ls/home/  /homee  &> a.txt
  
[root@localhost ~]# cat a.txt
ls: cannot access/homee: No such file or directory
  
/home/:
rm
```
还可以 `&` 来表示, & 表示等同于的意思
```
[root@localhost ~]# ls/home/  /homee  > d.txt 2>&1 #2标准错误输出等同于1标准输出
  
[root@localhost ~]# cat d.txt
ls: cannot access/homee: No such file or directory
/home/:
rm
  
[root@localhost ~]# ls/home/  /homee  2> d.txt >&2  #1标准输出等同于2标准错误输出
```

### tee
功能说明：读取标准输出的数据，并将其内容输入成文件。 `>` 屏幕上不会同时输出
```
[root@localhost ~]# ps -aux | grep vim  | tee a.txt
root       3597 0.0  0.1 151456  5232 pts/1   S+   21:20   0:00 vim a.txt
root       3723 0.0  0.0 112640   956 pts/0   S+   21:29   0:00 grep --color=auto vim
  
[root@localhost ~]# cat a.txt
root       3597 0.0  0.1 151456  5232 pts/1   S+   21:20   0:00 vim a.txt
root       3723 0.0  0.0 112640   956 pts/0   S+   21:29   0:00 grep --color=auto vim
```

### 输入重定向
`<` `<<` 
`<<`[标志] 代表以[标志]结束

`wc` 统计
```
sam:/home/sam # wc < /etc/passwd
  32   77 1754
```

向a.txt 输入， 以 `EOF` 结束， `EOF` 可以替换成其他符号
```
[root@localhost ~]# cat > a.txt <<EOF
  >**********************************
  > ****  welcome to linux ****
  >**********************************
  > EOF
```

## 管道｜
管道 ： 前一个程序的标准输出，交给后一个程序做标准输入。
管道符 `|` ： `|`左边程序的标准输出，交给`|`右边程序做标准输入；可以多个 `|` 链接。
```
[root@localhost ~]# ps -aux | grep vim
```

## 文件查找
常用查找命令：
`which` ： 查看可执行文件的位置， 比 `whereis`更常用
`whereis` ：    查看可执行文件的位置 及相关文件
`grep` ：     过滤
`locate` ：      配合数据库缓存，快速查看文件位置，速度快，不够准确 ； 新文件`update`后才能查到
`find` ：        实际搜寻硬盘查询文件名称

### which
* 作用： 查看可执行文件的位置
* 用法： which COMMAND

### whereis
* 作用：  查看可执行文件的位置 及相关文件
* 用法： whereis COMMAND 

### locate
* 作用： 配合数据库缓存，快速查看文件位置 ， 新文件需要 `update`加入 数据库
* 用法： 

### grep
* 作用： 过滤文件内容
* 用法：grep [选项]... PATTERN [FILE]
  常用选项： 
    `-v`:  反转
    `-i`:  忽略大小写
  常用PATTERN： 
    `^#`:   以#开头
    `#$`:   以#结尾
    `^$`:   空行
* 例子
```
#-v取反，查找出文件中不带nologin的行
[root@localhost ~]# grep  -v "nologin"   passwd
  
# -i  忽略大小写进行查找
[root@localhost ~]# grep -i "rm"   passwd 
  
# ^ 过滤文件中的所有以#号开头的行
[root@localhost ~]#grep "^#"  passwd 
  
# `$` 过滤文件中的所有以%结尾的行
[root@localhost ~]#grep "%$"  passwd 
  
# ^$ 过滤文件中的空行， -n 对过滤的内容加上行号
[root@localhost ~]#grep "^$"  passwd  -n  
```

#### 扩展：过滤真实有效的内容
```
[root@localhost ~]#grep  -v "^$"  /etc/ssh/sshd_config | grep  -v "^#"
```

### find
* 作用： 实际搜寻硬盘查询文件名称
* 用法： find [路径...] [条件] [动作]

#### [路径...]
不输入默认是当前目录
可以多个目录
```
[root@localhost test]# find t1/ tt1/ -type d
```
<br><br>
#### [条件]
##### 用户和组：-user　-group
例：查找home目录下所有的属于sam的文件
```
[root@localhost ~]# find /home/  -user "sam"
```

##### 类型：-type
`type` :  f 文件 ， d 目录 ， l 连接 ， p 管道 ，c 字符文件 ，b 块文件，s socket文件 
```
[root@localhost ~]# find /home/ -type f
[root@localhost ~]# find /home/ -type d
```

##### 名字：-name
`*` 通配符
```
[root@localhost ~]# find /home/ -name '*user*'
```

##### 大小：-size
`+NM` 大于N兆   `-NG` 小于ＮGB
```
[root@localhost ~]# find /boot/ -size +2M
```
 

##### 时间：*time | *min
[`-mtime`  `-atime`  `-ctime` | `-mmin` `-amin` `-cmin`]   n
`*time` 以24小时为单位， `*min`以分钟为单位; 
`+n`: 大于n， `n`：等于n， `-n`：小于n
```
[root@localhost ~]# date-s  2015-12-11
Fri Dec 11 00:00:00 CST 2015
  
[root@localhost ~]# find /root/  -mtime +1 | grep time.txt
/root/time.txt
  
[root@localhost ~]# find /root/  -mtime 2 | grep time.txt
/root/time.txt
```
 

##### 权限：-perm
```
[root@localhost ~]# find /boot/ -perm 755                  #等于0775权限的文件或目录
  
  # super auth 4   SUID  2 SGID   1  sticky
[root@localhost ~]# find /tmp/-perm -777                  #至少有777权限的文件或目录
```
 
例：
```
[root@localhost ~]# mkdir ccc
[root@localhost ~]# chmod 777 ccc
[root@localhost ~]# mkdir test
[root@localhost ~]# chmod 1777 test/
[root@localhost ~]# touch b.sh
[root@localhost ~]# chmod 4777 b.sh
  
  ## 查找：
[root@localhost ~]# find /root/  -perm 777
/root/ccc
  
[root@localhost ~]# find /root/ -perm 1777
/root/test
  
[root@localhost ~]# find /root/ -perm 4777
/root/b.sh
  
[root@localhost ~]# find /root/ -perm -777
/root/ccc
/root/test
/root/b.sh
```

##### 目录深度：-maxdepth
```
[root@localhost ~]# find /boot/ -maxdepth 2              #只查找目录第二层的文件和目录
```
 

##### 多条件组合：
`-a`  `-o`  `!`   或  `-and`  `-or` `-not`
```
[root@localhost ~]#find -type f -a -perm -002
[root@localhost ~]#find -type f -and -perm /o+w
./b.sh
[root@localhost ~]# find! -type f -and -perm -001
```
 

#### [动作]
`-ls` `-ok` `-exec` `-print` `-printf`
常用 `-exec`
```
[root@localhost ~]#find /root/test2/  -type f -exec rm {} \;
[root@localhost ~]# ls test2/
boot
[root@localhost ~]#find /root/test2/  -type f | xargs rm -rf
[root@localhost ~]# ls test2/
boot
  
[root@localhost findresults]# find /  -user sam   -exec cp -ra {} /findresults/ \;
```
参数解释：
 `-exec`： 执行命令 
 `rm`： 要执行的命令
 `{}`： 表示 `find` 查找出来了文件内容
 `\;`： `{}` 和 `\;` 之间要有空格。 固定语法，就是以这个结尾
 
#### 组合使用
```
find / -name "40*" -a -type f | xargs  grep " not found"
```
 
## 扩展：
### linux中ctime,mtime,atime
#### 区别
`atime` ：“访问时间(accesstime)”
`ctime` ：“改变时间(changetime)”
`mtime` ：“修改时间(modificationtime)”
改变和修改之间的区别在于是改文件的属性还是更改它的内容， *min类似。
`chmod a-w myfile`  这是一个属性改变；改变的ctime
`echo  aaa > bajie` 这是一个内容修改；同时修改mtime和ctime。

例如：
```
[root@localhost ~]# touch time.txt
[root@localhost ~]# stat time.txt
  File: ‘time.txt’
  Size: 0           Blocks:0          IO Block: 4096   regular empty file
Device: fd01h/64769d  Inode: 35698271    Links: 1
Access: (0644/-rw-r--r--)  Uid: (   0/    root)   Gid: (   0/    root)
Access: 2015-12-0820:59:28.293035474 +0800
Modify: 2015-12-0820:59:28.293035474 +0800
Change: 2015-12-0820:59:28.293035474 +0800
```
```
[root@localhost ~]#chmod u+x time.txt
[root@localhost ~]#stat time.txt
  File: ‘time.txt’
  Size: 0           Blocks:0          IO Block: 4096   regular empty file
Device: fd01h/64769d  Inode: 35698271    Links: 1
Access:(0744/-rwxr--r--)  Uid: (    0/   root)   Gid: (    0/   root)
Access: 2015-12-0820:59:28.293035474 +0800
Modify: 2015-12-0820:59:28.293035474 +0800
Change: 2015-12-08 21:02:06.612034204 +0800
```
可以看到，文件的ctime发生了改变

```
[root@localhost ~]# echo "aaaaaaaaaaa" >> time.txt
[root@localhost ~]# stat time.txt
  File: ‘time.txt’
  Size: 4           Blocks:8          IO Block: 4096   regular file
Device: fd01h/64769d  Inode: 35698271    Links: 1
access: (0744/-rwxr--r--)  Uid: (   0/    root)   Gid: (   0/    root)
Access: 2015-12-0820:59:28.293035474 +0800
Modify: 2015-12-08 21:03:41.775033441 +0800
Change: 2015-12-08 21:03:41.775033441 +0800
```
可以看到文件的ctime和mtime发生了改变
结论：无论是文件的属性还是内容发生变化，文件的ctime都会发生改变，只有文件内容发生修改的时候，文件的mtime才会发生变化

 
```
[root@localhost ~]# cat time.txt
[root@localhost ~]# stattime.txt
  File: ‘time.txt’
  Size: 4           Blocks:8          IO Block: 4096   regular file
Device: fd01h/64769d  Inode: 35698271    Links: 1
Access: (0744/-rwxr--r--)  Uid: (   0/    root)   Gid: (   0/    root)
Access: 2015-12-0821:06:45.223031969 +0800
Modify: 2015-12-0821:03:41.775033441 +0800
Change: 2015-12-0821:03:41.775033441 +0800
```
可以看到文件的atime发生了改变
结论：只要对文件内容进行了查看，那么atime就会发生改变

 
#### 显示
ls(1) 命令可用来列出文件的 atime、ctime和 mtime。
ls -lc filename         列出文件的 ctime     ll  -c
ls -lu filename         列出文件的 atime          ll  -u
ls -l filename          列出文件的 mtime     ll

  
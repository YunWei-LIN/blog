title: 1-29 Linux 正则表达式 sed awk cut
date: 2016-1-6 19:59:44
tags: GOD linux
categories: linux
---

主要内容
* 正则表达式 
* sed 
* awk 
* cut

<!-- more -->

## 正则表达式 
正则表达式，又称正规表示法、常规表示法（英语：Regular Expression，在代码中常简写为regex、regexp或RE），计算机科学的一个概念。正则表达式使用单个字符串来描述、匹配一系列符合某个句法规则的字符串。
正则表达式是一种符号表示法，被用来识别文本模式。

### grep
grep : “global regular expression print”，所以我们能看出 grep 程序和正则表达式有关联。 本质上，grep 程序会在文本文件中查找一个指定的正则表达式，并把匹配行输出到标准输出。
参数：

    -i 忽略大小写
    -v 反转
    -n  显示行号

### 元字符
    ^ $ . [ ] { } - ? * + ( ) | \
其他都被认为是原义字符； 元字符可以通过 `\` 来转义为原义字符。

### 锚点 ^ $
* `^` 行首
* `$` 行尾
* `\<` 表示词首  
* `\>` 表示词尾

^word：待搜寻的字符串(word)在行首

```
[root@localhost ~]# grep "^sync" /etc/passwd
sync:x:5:0:sync:/sbin:/bin/sync
```

word$：待搜寻的字符串(word)在行尾

```
[root@localhost ~]# grep "shutdown$" passwd
shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
```

显示空白行及行号

```
[root@localhost ~]# grep "^$" passwd  -n
50:
```

精确匹配 ： `'\<匹配内容\>'` 

```
[root@localhost ~]# ifconfig |grep '\<inet\>'
        inet 192.168.1.68  netmask 255.255.255.0  broadcast 192.168.1.255
        inet 127.0.0.1  netmask 255.0.0.0
```

### 转义 \
搜寻包括单引号 '的行，并把行号也打印出来
```
[root@localhost ~]# grep \' passwd
halt:x:7:0:'halt':/sbin:/sbin/halt
'mail':x:8:12:mail:/var/spool/mail:/sbin/nologin
```

过滤文件的反斜杠‘’\“
```
[root@localhost ~]# grep '\\' passwd
gopher:x\:13:30:gopher:/var/gopher:/sbin/nologin
[root@localhost ~]# grep -F '\'  passwd
gopher:x\:13:30:gopher:/var/gopher:/sbin/nologin
[root@localhost ~]# fgrep '\' passwd
gopher:x\:13:30:gopher:/var/gopher:/sbin/nologin
```

### 字符集合 [ ]
* `[list]` ：字符集合，里面列出想要选择的字符

```
[root@localhost ~]# grep g[ao] passwd
games:x:12:100:games:/usr/games:/sbin/nologin
gopher:x\:13:30:gopher:/var/gopher:/sbin/nologin
```

* [n1-n2]：字符集合的，里面列出想要包括的字符范围！

搜寻含有数字的3到5的行：
```
[root@localhost ~]# grep [3-5] passwd
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
sync:x:5:0:sync:/sbin:/bin/sync
```

开始为大写字母的：
```
[root@localhost ~]# grep ^[A-Z]  passwd
Rm2:x:502:500::/home/rm:/bin/bash
Rm3:x:503:500::/home/rm:/bin/bash
```

### 否定 [^ ]
`[^list]` : 否定字符集合，里面列出想要排除的字符, `^` 必须是 `[]` 内的第一个符合才能启动否定。

不想要开头是英文字母：
```
[root@localhost ~]# grep ^[^a-zA-Z] passwd
'mail':x:8:12:mail:/var/spool/mail:/sbin/nologin
"uucp":x:10:14:uucp:/var/spool/uucp:/sbin/nologin
#rmt:x:500:500::/home/rm:/bin/bash
```

### POSIX 字符集 
|字符集 | 说明|
|--------|---------------------------------------------------|
|[:alnum:] | 字母数字字符。在 ASCII 中，等价于：[A-Za-z0-9]|
|[:word:] | 与[:alnum:]相同, 但增加了下划线字符。|
|[:alpha:] | 字母字符。在 ASCII 中，等价于：[A-Za-z]|
|[:blank:] | 包含空格和 tab 字符。|
|[:cntrl:] | ASCII 的控制码。包含了0到31，和127的 ASCII 字符。|
|[:digit:] | 数字0到9|
|[:graph:] | 可视字符。在 ASCII 中，它包含33到126的字符。|
|[:lower:] | 小写字母。|
|[:punct:] | 标点符号字符。在 ASCII 中，等价于：|
|[:print:] | 可打印的字符。在[:graph:]中的所有字符，再加上空格字符。|
|[:space:] | 空白字符，包括空格，tab，回车，换行，vertical tab, 和 form feed.在 ASCII 中， 等价于：[ \t\r\n\v\f]|
|[:upper:] | 大写字母。|
|[:xdigit:] | 用来表示十六进制数字的字符。在 ASCII 中，等价于：[0-9A-Fa-f] |

### 限定符
`.`  代表绝对有一个任意字符的意思；
`*`  代表重复前一个字符到无穷次的意思;
`.*` 任长度的字符。

寻找包括有r开头和t结束且长度为四个字符行：
```
[root@localhost ~]# grep r..t passwd
root:x:0:0:root:/root:/bin/bash
operator:x:11:0:"operator":/root:/sbin/nologin
ftp:x:14:50:FTP User:/var/ftp:/sbin/nologin
```

寻找oo, ooo, oooo 等等的数据,也就是说,至少要有两个o 以上：
```
[root@localhost ~]# grep ooo* passwd
root:x:0:0:root:/root:/bin/bash
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
```

寻找包括g开头和g结束的字符串,中间可有可无
```
[root@localhost ~]# grep ^g.*g  passwd
```

#### 扩展
`\?` 用于修饰前导字符，表示前导字符出现0或1次, 例： grep "test\?" 
`\+` 用于修饰前导字符，表示前导字符出现1或多次   例：a\+ 匹配1或多个a
`\{n,m\}`  用于修饰前导字符，表示前导字符出现n至m次 （n和m都是整数，且n<m）
例：a\{3,5\} 匹配3至5个连续的a










## sed 
* 概念
`sed` : strem editor  流编辑器
sed编辑器是一行一行的处理文件内容的。正在处理的内容存放在模式空间(缓冲区)内，处理完成后按照选项的规定进行输出或文件的修改。它一次处理一行内容。处理时，把当前处理的行存储在临时缓冲区中，称为“模式空间”（pattern space），接着用sed命令处理缓冲区中的内容，处理完成后，把缓冲区的内容送往屏幕。接着处理下一行，这样不断重复，直到文件末尾。文件内容并没有 改变，除非你使用重定向存储输出。Sed主要用来自动编辑一个或多个文件；简化对文件的反复操作；

* 用法

      sed  [options]  ‘[command]’  [filename]
options:
`-n`	抑制自动(默认的) 输出   读取下一个输入行  ***常用***
-e	执行多个sed指令
-f	运行脚本
`-i`	编辑文件内容 ***常用***
-i.bak	编辑的同时创造.bak的备份
-r	使用扩展的正则表达式 
  
  command:
  `a`	在匹配后追加  ***常用***
  `i`	在匹配前插入  ***常用***
  `p`	打印  ***常用***
  `d`	删除  ***常用***
  r/R	读取文件/一行
  w	另存
  s	查找
  c	替换
  y	替换
  `h/H`	复制拷贝/追加模式空间(缓冲区)到存放空间  ***常用***
  `g/G`	粘贴 从存放空间取回/追加到模式空间       ***常用***
  x	两个空间内容的交换
  n/N	拷贝/追加下一行内容到当前
  D	删除\n之前的内容
  P	打印\n之前的内容
  b	无条件跳转
  t	满足匹配后的跳转
  T	不满足匹配时跳转


### s 替换
```
[root@localhost ~]# sed 's/root/ROOT/' /etc/passwd > newfilename
```
* s/regexp/replacement/

* /../../	分隔符
分割符 "/" 可以用别的符号代替 , 比如 “,”   “|”   “_“等

* 用 & 表示匹配的字符串
对文件中的root内容两边加上（）

```
[root@localhost ~]# sed 's/root/(&)/'  passwd
```

对文件中的所有以小写字母开头的行进行注释
```
[root@localhost ~]# sed 's/^[a-z]/#&/' passwd
```

sed 默认只替换搜索字符串的第一次出现 , 利用 /g 可以替换搜索字符串所有
```
[root@localhost ~]# sed -e '3,5s/nologin/bash/' -e '9,11s/sbin/bin/' passwd
```

### p 打印
* 显示文件第三行

```
[root@localhost ~]# sed -n '3p' passwd
roott
```

* 显示文件前三行
```
[root@localhost ~]# sed -n '1,3p' passwd
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
roott
```

* 显示文件除前三行之外的全部内容
```
[root@localhost ~]# sed -n '1,3!p' passwd
```

* 显示文件第三行和之后的三行
```
[root@localhost ~]# sed -n '3,+3p' passwd
roott
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
```

### i 匹配前插入
* 在文件头插入“###”
```
[root@localhost ~]# sed '1i###' passwd  > a.txt
```

### a 在匹配后追加
* 在文件尾插入"@@@"
```
[root@localhost ~]# sed '$a@@@' passwd
```

* 在文件的第二行后插入两行内容
```
[root@localhost ~]# sed '2a hello \
how are you?  \
what is your name?' passwd
```

### c 替换
* 把文件第三行替换成“$$$”
```
[root@localhost ~]# sed '3c$$$' passwd
```

### 复制粘贴
`h/H`	复制拷贝/追加模式空间(缓冲区)到存放空间
`g/G`	粘贴 从存放空间取回/追加到模式空间

* 把文件的第二行到第四行复制到文件的末尾
```
[root@localhost ~]# sed '2,4H;$G' passwd  > b.txt
```

### d 删除 
* 删除空行    
[root@localhost ~]# sed '/^$/d' passwd
把fstab中包含xfs的记录（行）写入新的文件中
[root@localhost ~]# sed '/xfs/w newfstab2' /etc/fstab


### 直接修改文件
sed 的-i选项可以直接修改文件中的内容
```
[root@localhost ~]# sed -i 's/root/rm/' passwd
```


### 参考
[sed 简明教程](http://coolshell.cn/articles/9104.html)


## awk 
`AWK` 是一种优良的文本处理工具，Linux及Unix环境中现有的功能最强大的数据处理引擎之一。这种编程及数据操作语言的最大功能取决于一个人所拥有的知识。awk命名:Alfred Aho Peter Weinberger和brian kernighan三个人的姓的缩写。
最简单地说， AWK 是一种用于处理文本的编程语言工具。
任何awk语句都是由模式和动作组成，一个awk脚本可以有多个语句。模式决定动作语句的触发条件和触发时间。

### 基本语法

      netstat | awk '{print $1, $7}'

####  起步
单引号中的被大括号括着的就是awk的语句，注意，其只能被单引号包含。
其中的$1..$n表示第几例。注：$0表示整个行。

#### 保留字
`BEGIN` ：在任何动作之前进行。
`END` ： 在完成动作之后执行。

#### 格式化输出
print

#### 过滤记录
* 比较运算符
==，!=, >, <, >=, <=

* 逻辑运算符
&& ， ||

#### 内建变量
|$0 | 当前记录（这个变量中存放着整个行的内容）|
|------|------------------------------------------------|
|$1~$n | 当前记录的第n个字段，字段间由FS分隔|
|FS | 输入字段分隔符 默认是空格或Tab|
|NF | 当前记录中的字段个数，就是有多少列|
|NR | 已经读出的记录数，就是行号，从1开始，如果有多个文件话，这个值也是不断累加中。|
|FNR | 当前记录数，与NR不同的是，这个值会是各个文件自己的行号|
|RS | 输入的记录分隔符， 默认为换行符|
|OFS | 输出字段分隔符， 默认也是空格|
|ORS | 输出的记录分隔符，默认为换行符|
|FILENAME | 当前输入文件的名字|


#### 指定分隔符
默认分隔符是空格或Tab

指定冒号分割

      awk  -F: '{print $1,$3,$6}' /etc/passwd
      或
      awk  'BEGIN{FS=":"} {print $1,$3,$6}' /etc/passwd

指定多个分隔符

      awk -F '[;:]'

#### 字符串匹配
匹配FIN状态，其实 ~ 表示模式开始。/ /中是模式。这就是一个正则表达式的匹配。 `|| NR==1` 其实是不处理第一行。

      awk '$6 ~ /FIN/ || NR==1 {print NR,$4,$5,$6}' OFS="\t" netstat.txt

模式取反

      awk '$6 !~ /WAIT/ || NR==1 {print NR,$4,$5,$6}' OFS="\t" netstat.txt
      或
      awk '!/WAIT/' netstat.txt

#### 折分文件
awk拆分文件很简单，就使用重定向。如按第6例分隔文件，相当的简单（其中的NR!=1表示不处理表头）。

    awk 'NR!=1{print > $6}' netstat.txt
    
你也可以把指定的列输出到文件

    awk 'NR!=1{print $4,$5 > $6}' netstat.txt


#### awk脚本
语法如下：

    BEGIN{ 这里面放的是执行前的语句 }
    END {这里面放的是处理完所有的行后要执行的语句 }
    {这里面放的是处理每一行时要执行的语句}

有这么一个文件（学生成绩表）： score.txt

    Marry   2143 78 84 77
    Jack    2321 66 78 45
    Tom     2122 48 77 71
    Mike    2537 87 97 95
    Bob     2415 40 57 62

awk脚本如下

    #!/bin/awk -f
    #运行前
    BEGIN {
	math = 0
	english = 0
	computer = 0
    
	printf "NAME    NO.   MATH  ENGLISH  COMPUTER   TOTAL\n"
	printf "---------------------------------------------\n"
    }
    #运行中
    {
	math+=$3
	english+=$4
	computer+=$5
	printf "%-6s %-6s %4d %8d %8d %8d\n", $1, $2, $3,$4,$5, $3+$4+$5
    }
    #运行后
    END {
	printf "---------------------------------------------\n"
	printf "  TOTAL:%10d %8d %8d \n", math, english, computer
	printf "AVERAGE:%10.2f %8.2f %8.2f\n", math/NR, english/NR, computer/NR
    }

执行

    awk -f cal.awk score.txt
    或
    ./cal.awk score.txt

### 实例
* 获取当前主机网卡的ip地址
  ```
  ipaddress=$(ifconfig | grep broadcast | awk '{print $2}')
  ```

* 打印出passwd中用户UID大于1000的用户名和登录shell
  ```
  [root@localhost ~]# cat /etc/passwd | awk -F: '$3>1000 {print $1 "\t" $7}'
  nfsnobody       /sbin/nologin
  rm1     /bin/bash
  rm2     /bin/bash
  rm3     /bin/bash
  ```

* 多条件进行过滤
能登录的普通用户

  ```
  [root@localhost ~]# cat /etc/passwd | awk -F: 'BEGIN {print "name \t shell"} $3>1000 && $7=="/bin/bash" {print $1 "\t" $7} END {print "Thank you!!!"}'
  name     shell
  rm1     /bin/bash
  rm2     /bin/bash
  rm3     /bin/bash
  Thank you!!!
  ```

* awk在脚本中的使用
找到系统中所有的能登录的普通用户，并进行删除
  
  ```
  #!/bin/bash
  user=$(cat /etc/passwd | awk -F: '$3>=1000 && $7=="/bin/bash" {print $1}')
  for i in $user
  do
	  userdel -r $i
  done
  ```

* RHEL7中查看系统当前内存使用百分比
  
  ```
  #!/bin/bash
  echo "此脚本可以用来查看当前内存使用百分比"
  use=$(free -m | grep "Mem:" | awk '{print $3}')
  total=$(free -m | grep "Mem:" | awk '{print $2}')
  useper=$(expr $use \* 100 / $total)
  echo "系统当前内存使用百分比为:"
  echo ${useper}%
  ```







## cut
cut是一个选取命令，就是将一段数据经过分析，取出我们想要的。一般来说，选取信息通常是针对“行”来进行分析的，并不是整篇信息分析的。
主要参数
-b ：以字节为单位进行分割。这些字节位置将忽略多字节字符边界，除非也指定了 -n 标志。
-c ：以字符为单位进行分割。
-d ：自定义分隔符，默认为制表符。
-f ：与-d一起使用，指定显示哪个区域。

* 以字节进行分割
```
[root@localhost ~]# tail -3 /etc/passwd | cut -b 1-4,11,12
post89
sshd74
tcpd72
```
* 以字符进行分割
```
[root@localhost ~]# cat newyear.txt | cut -c 1-4
```

* 以空格作为分隔符
```
[root@localhost ~]# sed -n l c.txt
aaaa\tbbbb    cccc$
aaaa    bbbb\tcccc$
[root@localhost ~]# cat c.txt  | cut -d ' ' -f 1
aaaa    bbbb
aaaa
```

总结：
cut有哪些缺陷和不足？
如果文件里面的某些域是由若干个空格来间隔的，那么用cut就有点麻烦了，因为cut只擅长处理“以一个字符间隔”的文本内容
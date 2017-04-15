title: shell （一） 基础
date: 2015-12-25 18:29:44
tags: shell
categories: linux
---

基本内容
* 元字符
* 变量
* 注释

<!-- more -->

## 元字符
具有特定功能的保留字
`IFS（交换字段分隔符）`：由<space> <tab> <enter>三者之一组成（我们常用space），用来拆分command line的每一个词（word）用的，因为shell command line是按词来处理的。
`CR（回车键）` ： 由<enter>产生，用来结束command line。
`=` ：  设定变量。
`$` ：  做变量或运算替换(请不要与 shell prompt 搞混了)。
`>` ：  重定向 stdout（标准输出standard out）。
`<` ：  重定向 stdin（标准输入standard in）。
`|`：   管道命令。
`&` ：  重定向 file descriptor （文件描述符），或将命令置于后台执行。
`( )`： 將其內的命令置于 nested subshell （嵌套的子shell）执行，或用于运算或命令替换。
`{ }`： 將其內的命令置于 non-named function（未命名函数） 中执行，或用在变量替换的界定范围。
`;` ：  在前一个命令结束时，而忽略其返回值，继续执行下一個命令。
`&&` ： 在前一個命令结束时，若返回值为 true，继续执行下一個命令。
`||` ： 在前一個命令结束时，若返回值为 false，继续执行下一個命令。
`!`：   执行 history 列表中的命令
`;` : 分隔同一行的2条shell语句。





## 变量
临时变量：是shell程序内部定义的，其使用范围仅限于定义它的程序，对其它程序不可见。
环境变量(永久变量): 其值不随shell 脚本的执行结束而消失。


### 自定义环境变量
编辑用户配置文件，利用export修改环境变量
```
[root@localhost ~]#export PATH=/root:$PATH
source 配置文件 #及时生效
```

### 临时变量
由字母或下划线打头。 由字母、数字或下划线组成，并且大小写字母意义不同。变量名长度没有限制。
使用变量值时，要在变量名前加上“$”

#### 变量赋值
赋值号“=”两边应没有空格。
```
[root@localhost ~]# abc = 123
bash: abc: command not found...
  
[root@localhost ~]# abc=123
```

#### 命令赋给变量
两种方式：
* ` 反引号 `

```
[root@localhost ~]#A=`date`
[root@localhost ~]#echo $A
Thu Dec 24 21:14:24 CST2015
```

* `$()`

```
[root@localhost ~]# B=$(date)
[root@localhost ~]# echo $B
Thu Dec 24 21:17:48 CST2015
```

#### 单引号和双引号的区别
单引号之间的内容成为纯文本，取消所有特殊符号的意义；
双引号仅特殊符号 **$ \ `** 这三个的含义保留。
```
[root@localhost ~]#NAME="rm  mk  docker"
[root@localhost ~]#echo $NAME
rm mk docker
  
[root@localhost ~]#NAME='rm  mk  docker'
[root@localhost ~]#echo $NAME
rm mk docker
  
[root@localhost ~]#NAME="rm $NAME"
[root@localhost ~]#echo $NAME
rm rm mk docker
  
[root@localhost ~]#NAME='rm $NAME'
[root@localhost ~]#echo $NAME
rm $NAME
```

#### unset readonly
```
unset variable_name #删除变量
readonly variable_name #只读变量
```

### 特殊变量
|变量|含义|
|------|---------------------------------------------|
|$0|当前脚本的文件名|
|$n|传递给脚本或函数的参数。n 是一个数字，表示第几个参数。例如，第一个参数是$1，第二个参数是$2。|
|!$|上个命令最后一个参数|
|$#|传递给脚本或函数的参数个数。|
|$*|传递给脚本或函数的所有参数。|
|$@|传递给脚本或函数的所有参数。被双引号(" ")包含时，与 $* 稍有不同，下面将会讲到。|
|$?|上个命令的退出状态，或函数的返回值。|
|$$|当前Shell进程ID。对于 Shell 脚本，就是这些脚本所在的进程ID。|
|$!|执行上一个后台程序的PID|
`$*` 和 `$@` 都表示传递给函数或脚本的所有参数，不被双引号(" ")包含时，都以"$1" "$2" … "$n" 的形式输出所有参数。
但是当它们被双引号(" ")包含时，`$*` 会将所有的参数作为一个整体，以"$1 $2 … $n"的形式输出所有参数；`$@` 会将各个参数分开，以"$1" "$2" … "$n" 的形式输出所有参数。

###   常用PATTERN： 
    ^#:   以#开头
    #$:   以#结尾
    ^$:   空行
    
### Read命令：
作用：从键盘读入数据，赋给变量
参数 ：-p    -n   -t参数结合使用
-p(提示语句)
-n(字符个数)
-t(等待时间)
```bash
[root@localhost ~]# vim read1.sh
#!/bin/bash
read -p  "请输入你的姓名："  YYNAME
read -p  "请输入你的年龄："  YYAGE
read -p  "请输入你的性别："  YYSEX
  
echo "你的基本信息如下："
echo "姓名：$YYNAME"
echo "年龄：$YYAGE"
echo "性别：$YYSEX"
```

### expr 命令

作用：Shell变量的算术运算：
expr命令：对整数型变量进行算术运算
语法： expr  表达式    #注意 运算符之间要有空格
```
[root@localhost ~]#expr 3 + 5
8
  
[root@localhost ~]#expr 8 - 5
3
  
[root@localhost ~]#expr 8 / 4
2
  
[root@localhost ~]#expr 8 \* 4
32
```
`注意：乘号(*)前边必须加反斜杠(\)才能实现乘法运算`


## 注释
以“#”开头的行就是注释，会被解释器忽略。
没有多行注释，只能每一行加一个#号。
```bash
##### 用户配置区 开始 #####
#
#
# 项目根目录
#
##### 用户配置区 结束  #####
```

遇到大段的代码需要临时注释起来，可以把这一段要注释的代码用一对花括号括起来，定义成一个函数，没有地方调用这个函数，这块代码就不会执行，达到了和注释一样的效果。 
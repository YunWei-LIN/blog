title: shell （二） 替换
date: 2015-12-28 18:29:44
tags: shell
categories: linux
---

主要内容：
* 转义符
* 命令替换
* 变量替换

<!-- more -->

## 转义符
如果表达式中包含特殊字符，Shell 将会进行替换，转义字符也是一种替换。
下面的转义字符都可以用在 echo 中：

|转义字符|含义|
|------|------------------------------|
|\\|反斜杠|
|\a|警报，响铃|
|\b|退格（删除键）|
|\f|换页(FF)，将当前位置移到下页开头|
|\n|换行|
|\r|回车|
|\t|水平制表符（tab键）| 
|\v|垂直制表符|

```bash
╭─sam@sam  ~  
╰─$ echo -E "Value of a is \n nnnnnnnnnnn "
Value of a is \n nnnnnnnnnnn 
  
╭─sam@sam  ~  
╰─$ echo  "Value of a is \n nnnnnnnnnnn " 
Value of a is 
nnnnnnnnnnn 
  
╭─sam@sam  ~  
╰─$ echo -e "Value of a is \n nnnnnnnnnnn "
Value of a is 
nnnnnnnnnnn 
```

可以使用 echo 命令的 -E 选项禁止转义， -e 选项是转义；`默认是不转义的`;
在没有 -E 的情况下,可承认并可以内置替换以下序列:

        \NNN  字符的ASCII代码为NNN(八进制)
        \\    反斜线
        \a    报警符(BEL)
        \b    退格符
        \c    禁止尾随的换行符
        \f    换页符
        \n    换行符
        \r    回车符
        \t    水平制表符
        \v    纵向制表符












## 命令替换

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

















## 变量替换

变量替换可以根据变量的状态（是否为空、是否定义等）来改变它的值

可以使用的变量替换形式：

|形式|说明|
|-------------|------------------------------------------------------------------------------------------------------|
|${var}|变量本来的值|
|${var:-word}|如果变量 var 为空或已被删除(unset)，那么返回 word，但不改变 var 的值。|
|${var:+word}|如果变量 var 被定义，              那么返回 word，但不改变 var 的值。|
|${var:=word}|如果变量 var 为空或已被删除(unset)，那么返回 word，并将 var 的值 `设置` 为 word。|
|${var:?message}|如果变量 var 为空或已被删除(unset)，那么将消息 message 送到 `标准错误输出`，可以用来检测变量 var 是否可以被正常赋值。 若此替换出现在Shell脚本中，那么脚本将停止运行。|































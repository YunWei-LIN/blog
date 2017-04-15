title: shell （七） 循环
date: 2015-12-28 21:49:44
tags: shell
categories: linux
---

主要内容

* for循环
* while循环
* until循环
* 跳出循环
<!-- more -->


## for循环
for循环一般格式为：

    for 变量 in 列表
    do
	command1
	command2
	...
	commandN
    done

列表是一组值（数字、字符串等）组成的序列，每个值通过空格分隔。每循环一次，就将列表中的下一个值赋给变量。
`in` 列表是可选的，如果不用它，for 循环使用命令行的位置参数。

* 例一
顺序输出当前列表中的数字：

```
for loop in 1 2 3 4 5
do
    echo "The value is: $loop"
done
```
    
* 例二
顺序输出字符串中的字符：

```
for str in 'This is a string'
do
    echo $str
done
```
    
* 例三
显示主目录下以 .bash 开头的文件：

```
#!/bin/bash
for FILE in $HOME/.bash*
do
    echo $FILE
done
```


## while循环
while循环用于不断执行一系列命令，也用于从输入文件中读取数据；命令通常为测试条件，条件为真时继续循环。其格式为：

    while command
    do
      Statement(s) to be executed if command is true
    done


* 例一 
COUNTER计数

```
COUNTER=0
while [ $COUNTER -lt 5 ]
do
    COUNTER='expr $COUNTER+1'
    echo $COUNTER
done
```


* 例二
读取键盘信息，按<Ctrl-D>结束循环。

```
echo 'type <CTRL-D> to terminate'
echo -n 'enter your most liked film: '
while read FILM
do
    echo "Yeah! great film the $FILM"
done
```





## until循环
until 循环执行一系列命令直至条件为 true 时停止。until 循环与 while 循环在处理方式上刚好相反。
command 一般为条件表达式，如果返回值为 false，则继续执行循环体内的语句，否则跳出循环。
until 循环格式为：

    until command
    do
      Statement(s) to be executed until command is true
    done


* 例
输出 0 ~ 9 

```
a=0
until [ ! $a -lt 10 ]
do
   echo $a
   a=`expr $a + 1`
done
```











## 跳出循环
在循环过程中，有时候需要在未达到循环结束条件时强制跳出循环，Shell也使用 break 和 continue 来跳出循环。 

### break
break命令允许跳出本层所有循环（终止执行后面的所有循环）

`break n` 表示跳出第 n 层循环。

例如：
```bash
    for var1 in 1 2 3
    do
       for var2 in 0 5
       do
          if [ $var1 -eq 2 -a $var2 -eq 0 ]
          then
             break 2
          else
             echo "$var1 $var2"
          fi
       done
    done
```
如上，break 2 表示直接跳出外层循环。 


### continue
continue命令与break命令类似，只有一点差别，它不会跳出所有循环，仅仅跳出当前循环。
`continue n` 表示跳出第 n 层循环。
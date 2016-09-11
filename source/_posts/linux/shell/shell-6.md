title: shell （六） 条件
date: 2015-12-28 21:29:44
tags: shell
categories: linux
---
主要内容
* if ... else 语句
* case ... esac 语句
* 

<!-- more -->


## if ... else 语句
Shell 有三种 if ... else 语句：

    if ... fi 语句；
    if ... else ... fi 语句；
    if ... elif ... else ... fi 语句。

### if ... fi
`if ... fi` 语句的语法：

    if [ expression ]
    then
      Statement(s) to be executed if expression is true
    fi

注意：expression 和方括号([ ])之间必须有空格，否则会有语法错误。

例如：

```bash
#!/bin/sh
a=10
b=20
if [ $a == $b ]
then
    echo "a is equal to b"
fi
if [ $a != $b ]
then
    echo "a is not equal to b"
fi
```

### if ... else ... fi 语句
`if ... else ... fi` 语句的语法：

    if [ expression ]
    then
      Statement(s) to be executed if expression is true
    else
      Statement(s) to be executed if expression is not true
    fi

例如：
```bash
#!/bin/sh
a=10
b=20
  
if [ $a == $b ]
then
    echo "a is equal to b"
else
    echo "a is not equal to b"
fi
```


### if ... elif ... fi 语句
`if ... elif ... fi` 语句可以对多个条件进行判断，语法为：

    if [ expression 1 ]
    then
      Statement(s) to be executed if expression 1 is true
      
    elif [ expression 2 ]
    then
      Statement(s) to be executed if expression 2 is true
    
    elif [ expression 3 ]
    then
      Statement(s) to be executed if expression 3 is true
    
    else
      Statement(s) to be executed if no expression is true
    fi

哪一个 expression 的值为 true，就执行哪个 expression 后面的语句；如果都为 false，那么不执行任何语句。

例如：
```bash
#!/bin/sh
a=10
b=20
  
if [ $a == $b ]
then
    echo "a is equal to b"
  
elif [ $a -gt $b ]
then
    echo "a is greater than b"
  
elif [ $a -lt $b ]
then
    echo "a is less than b"
  
else
    echo "None of the condition met"
fi
```

### test
if ... else 语句也经常与 test 命令结合使用， test 命令用于检查某个条件是否成立，与方括号([ ])类似。

```bash
if test $[a] -eq $[b]
then
    echo 'The two numbers are equal!'
else
    echo 'The two numbers are not equal!'
fi
```









## case ... esac 语句
`case ... esac` 是一种多分枝选择结构。case 语句匹配一个值或一个模式，如果匹配成功，执行相匹配的命令。
case语句格式如下：

    case 值 in
    模式1)
	command1
	command2
	command3
	;;
	
    模式2)
	command1
	command2
	command3
	;;
	
    *)
	command1
	command2
	command3
	;;
    esac

取值后面必须为关键字 in，每一模式必须以右括号结束。取值可以为变量或常数。匹配发现取值符合某一模式后，其间所有命令开始执行直至 ;;。
取值将检测匹配的每一个模式。一旦模式匹配，则执行完匹配模式相应命令后不再继续其他模式。如果无一匹配模式，使用星号 * 捕获该值，再执行后面的命令。

例如：
```bash
echo 'Input a number between 1 to 4'
echo 'Your number is:\c'
read num
case $num in
    1)  echo 'You select 1'
    ;;
    2)  echo 'You select 2'
    ;;
    3)  echo 'You select 3'
    ;;
    4)  echo 'You select 4'
    ;;
    *)  echo 'You do not select a number between 1 to 4'
    ;;
esac
```

```bash
option="${1}"
  
case ${option} in
  
   -f) FILE="${2}"
      echo "File name is $FILE"
      ;;
   -d) DIR="${2}"
      echo "Dir name is $DIR"
      ;;
   *) 
      echo "`basename ${0}`:usage: [-f file] | [-d directory]"
      exit 1 # Command to come out of the program with status 1
      ;;
esac
```


##  扩展 (())

条件测试使用 `[]` 时候，必须保证运算符与算数之间有空格。四则运算也只能借助：expr命令完成。 双括号 `(())` 结构语句，可以扩展shell中算数及赋值运算。

 
使用方法：

语法：

（（表达式1,表达式2…））

特点：

1、在双括号结构中，所有表达式可以像c语言一样，如：a++,b--等。 a=a+1

2、在双括号结构中，所有变量可以不加入：“$”符号前缀。

3、双括号可以进行逻辑运算，四则运算

4、双括号结构 扩展了for，while,if条件测试运算

5、支持多个表达式运算，各个表达式之间用逗号“，”分开














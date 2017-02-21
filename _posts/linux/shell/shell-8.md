title: shell （八） 函数, 文件包含, 转码工具
date: 2015-12-28 22:29:44
tags: shell
categories: linux
---

主要内容

* 函数
* 文件包含
* 转码工具
<!-- more  -->

## 函数
函数可以将一个复杂功能划分成若干模块，让程序结构更加清晰，代码重复利用率更高。函数必须先定义后使用。

### 函数定义

定义格式如下：
    function function_name () {
	list of commands
	[ return value ]
    }

或

    function_name () {
	list of commands
	[ return value ]
    }

`function` 可以省略，清晰起见，还是建议加上。

### 函数返回值

* 函数可以显式增加return语句；如果不加，会将最后一条命令运行结果作为返回值。
* Shell 函数返回值只能是整数，一般用来表示函数执行成功与否，0表示成功，其他值表示失败。如果 return 其他数据，比如一个字符串，往往会得到错误提示：“numeric argument required”。
* 如果一定要让函数返回字符串，那么可以先定义一个变量，用来接收函数的计算结果，脚本在需要的时候访问这个变量来获得函数返回值。

例：
```
#!/bin/bash
funWithReturn(){
    echo "The function is to get the sum of two numbers..."
    echo -n "Input first number: "
    read aNum
    echo -n "Input another number: "
    read anotherNum
    echo "The two numbers are $aNum and $anotherNum !"
    return $(($aNum+$anotherNum))
}
funWithReturn
# Capture value returnd by last command
ret=$?
echo "The sum of two numbers is $ret !"
```

### 函数删除
删除函数也可以使用 unset 命令，不过要加上 .f 选项，如下所示：

    $unset .f function_name

### 函数调用
调用只需要给出函数名，不需要加括号。
```
# Define your function here
Hello () {
   echo "Url is http://giveme5.top"
}
# Invoke your function
Hello
```




### 函数参数
调用函数时可以向其传递参数。在函数体内部，通过 $n 的形式来获取参数的值，例如，$1表示第一个参数，$2表示第二个参数...
注意，当n>=10时，需要使用 `${n}` 来获取参数。 获取第十个参数需要 `${10}` ， $10 不能获取第十个参数。

例如：
```bash
#!/bin/bash
funWithParam(){
    echo "The value of the first parameter is $1 !"
    echo "The value of the second parameter is $2 !"
    echo "The value of the tenth parameter is $10 !"
    echo "The value of the tenth parameter is ${10} !"
    echo "The value of the eleventh parameter is ${11} !"
    echo "The amount of the parameters is $# !"  # 参数个数
    echo "The string of the parameters is $* !"  # 传递给函数的所有参数
}
funWithParam 1 2 3 4 5 6 7 8 9 34 73
```

输出：

    he value of the first parameter is 1 !
    The value of the second parameter is 2 !
    The value of the tenth parameter is 10 !
    The value of the tenth parameter is 34 !
    The value of the eleventh parameter is 73 !
    The amount of the parameters is 12 !
    The string of the parameters is 1 2 3 4 5 6 7 8 9 34 73 !"

另外：
几个特殊变量用来处理参数（[更多特殊变量](/2015/12/25/linux/xuegodLinux/godLinux1-24-28-shell/godLinux1-24/#特殊变量)）：

|特殊变量|说明|
|----|----------------------------|
|$#|传递给函数的参数个数。|
|$*|显示所有传递给函数的参数。|
|$@|与$*相同，但是略有区别，请查看[更多特殊变量](/2015/12/25/linux/xuegodLinux/godLinux1-24-28-shell/godLinux1-24/#特殊变量)。|
|$?|函数的返回值。|

<br><br>
#### shift
shift: 参数左移指令, 每执行一次，参数序列顺次左移一个位置，$#的值减1，用于分别处理每个参数，移出去的参数，不再可用。
![](http://7xklqw.com1.z0.glb.clouddn.com/sh_shift.png)

例：
加法计算器，通过 `shift` 指令使参数左移，求出所有参数的和

```bash
#!/bin/bash
if [ $# -le 0 ]
then
	echo "err!:Not enough parameters"
exit  124
fi
sum=0
while [ $# -gt 0 ]
do
sum=`expr $sum + $1`
  
shift #参数左移
  
done
echo $sum
```


## 文件包含
文件包含即将外部脚本的内容合并到当前脚本。

可以使用：

    . filename

或

    source filename

两种方式的效果相同，简单起见，一般使用点号(.)，但是注意点号(.)和文件名中间有一空格。

例如：
创建两个脚本，一个是被调用脚本 subscript.sh，内容如下：
```
    url="http://see.xidian.edu.cn/cpp/view/2738.html"
```

一个是主文件 main.sh，内容如下：
```
    #!/bin/bash
    . ./subscript.sh
    echo $url
```

注意：被包含脚本不需要有执行权限。 


## dos2unix
由于编码问题，在windows中开发的脚本导入到Linux系统后执行报错，主要是因为在windows开发保存时没注意编码（UTF8）和换行符（LF）。
在Linux中可以用工具dos2unix解决。

* 安装
      [root@localhost test]#rpm -ivh /mnt/Packages/dos2unix-6.0.3-4.el7.x86_64.rpm

* 使用
      [root@localhost ]# dos2unix test.sh
      dos2unix: converting file test.sh to Unix format ...
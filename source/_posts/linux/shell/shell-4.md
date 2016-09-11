title: shell （四） 字符串
date: 2015-12-28 19:59:44
tags: shell
categories: linux
---

## 主要内容
* 字符串拼接
* 字符串长度
* 字符串截取
* 字符串查找


字符串是shell编程中最常用最有用的数据类型，字符串可以用单引号，也可以用双引号，也可以不用引号。

单引号和双引号的区别 : 
* 单引号之间的内容成为纯文本，取消所有特殊符号的意义；
* 双引号仅特殊符号 **$ \ `** 这三个的含义保留。
<!-- more -->


## 字符串拼接
```bash
name="sam"
greeting="hello, "$name" !"
greeting_1="hello, ${name} !"
```

## 字符串长度
```bash
string="abcd"
echo ${#string} # 4
```

## 字符串截取
```bash
string="Hello world, shell"
echo ${string:0:4} #输出Hell
echo ${string:1:4} #输出ello
echo ${string:1:1} #输出e
```

`${string:index:length}` , index 从 0 开始。

## 字符串查找
```
string="alibaba is a great company"
echo `expr index "$string" is` #3 ， i或s最早出现的地方
```

待找个更好的
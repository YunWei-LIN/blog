title: shell （五） 数组
date: 2015-12-28 20:29:44
tags: shell
categories: linux
---
主要内容
bash支持一维数组（不支持多维数组），并且没有限定数组的大小。数组元素的下标由0开始编号。获取数组中的元素要利用下标，下标可以是整数或算术表达式，其值应大于或等于0。

* 数组定义
* 数组读取
* 数组长度
<!-- more -->

## 数组定义
在Shell中，用括号来表示数组，数组元素用“空格”符号分割开。定义数组的一般形式为：
      array_name=(value1 ... valuen)

例如：
```
array_name=(value0 value1 value2 value3)
  
#或者
array_name=(
value0
value1
value2
value3
)
    
#或者    
array_name[0]=value0
array_name[1]=value1
array_name[2]=value2
#可以不使用连续的下标，而且下标的范围没有限制。 
```





## 数组读取
读取数组元素值的一般格式是：
      ${array_name[index]}


例如：
```
valuen=${array_name[2]}
```

使用 `@ 或 * ` 可以获取数组中的所有元素，例如：
```
${array_name[*]}
${array_name[@]}
```



## 数组长度
获取数组长度的方法与获取字符串长度的方法相同, ` # `

例如：
```
# 取得数组元素的个数
length=${#array_name[@]}
# 或者
length=${#array_name[*]}
# 取得数组单个元素的长度
lengthn=${#array_name[n]}
```
title: Java 8 内置函数式接口
date: 2017-10-17 19:44:58
tags: [java 8, jdk8, FunctionalInterface, 内置函数式接口]
categories: java
---

## 主要内容
函数式接口


*更新历史*
无

<!-- more -->

---

## 函数式接口
函数式接口就是只有一个方法的接口，如Runnable、Callable、Comparable都称作函数式接口。
java8专门新增 `FunctionalInterface` 来注解, 防止破坏函数式接口的定义； 默认方法和静态方法不会破坏函数式接口的定义。

## 内置函数式接口
java 8 之前 有 Runnable、Callable、Comparable 等； java8提供了一批内置的函数式接口， 跟 Lambda 表达式关系密切。

|接口|	参数|	返回类型| 抽象方法 |	说明|
|-----|----|----|-------|-----------|
|Predicate\<T\> |	T|boolean|test(T) -> boolean|	输入某个值，输出bool值，用于对某值进行判定。并可以使用 negate(非)， and 和 or 连接|
|Consumer\<T\> |	T	|void	|accept(T) -> void|输入某值，无输出。用于消费某值|
|Function<T,R> |	T	|R	|apply(T) -> R|输入某类型值，输出另种类型值，用于类型转化等|
|Supplier\<T\> |	None	|T	|get() -> T|无输入，输出某值，用于生产某值|
|UnaryOperator\<T\>	 |T	|T	|          |输入某类型值，输出同类型值，用于值的同类型转化，如对值进行四则运算等|
|BinaryOperator\<T\> |(T,T)|	T	|              |输入两个某类型值，输出一个同类型值，用于两值合并等|


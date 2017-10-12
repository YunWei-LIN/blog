title: Java 8 Streams
date: 2017-10-12 19:44:58
tags: [java 8, jdk8, Streams]
categories: java
---

## 主要内容
Java 8 Streams 在集合和其他数据集上运行函数式操作。
所有流计算都有一种共同的结构：它们具有一个流来源、0 或多个中间操作，以及一个终止操作。

*更新历史*
无

<!-- more -->

***



## 来源

| 方法 |               描述               |
| ------ | ------------------------ |
|Collection.stream() | 使用一个集合的元素创建一个流。|
|Stream.of(T...)|使用传递给工厂方法的参数创建一个流。|
|Stream.of(T[])|使用一个数组的元素创建一个流。|
|Stream.empty()|创建一个空流。|
|Stream.iterate(T first, BinaryOperator<T> f)|创建一个包含序列 first, f(first), f(f(first)), ... 的无限流|
|Stream.iterate(T first, Predicate<T> test, BinaryOperator<T> f)|（仅限 Java 9）类似于 Stream.iterate(T first, BinaryOperator<T> f)，但流在测试预期返回 false 的第一个元素上终止。|
|Stream.generate(Supplier<T> f)|使用一个生成器函数创建一个无限流。|
|IntStream.range(lower, upper)|创建一个由下限到上限（不含）之间的元素组成的 IntStream。|
|IntStream.rangeClosed(lower, upper)|创建一个由下限到上限（含）之间的元素组成的 IntStream。|
|BufferedReader.lines()|创建一个有来自 BufferedReader 的行组成的流。|
|BitSet.stream()|创建一个由 BitSet 中的设置位的索引组成的 IntStream。|
|Stream.chars()|创建一个与 String 中的字符对应的 IntStream。|


## 中间操作
负责将一个流转换为另一个流

|操作|内容|
| ------ | ------------------------ |
|filter(Predicate<T>)|接受一个断言（谓词，返回boolean的函数）作为参数，并返回一个包括所有符合断言的元素的流|
|map(Function<T, U>)|映射， 将提供的函数应用于流的元素，并将其映射成一个新的元素|
|flatMap(Function<T, Stream<U>>|映射，扁平， 将提供的函数应用于流元素后获得的流元素， 即将一个流中的每个值都换成另一个流，然后把所有的流连接成为一个流|
|distinct()|已删除了重复的流元素|
|sorted()|按自然顺序排序的流元素|
|Sorted(Comparator<T>)|按提供的比较符排序的流元素|
|limit(long)|截断至所提供长度的流元素|
|skip(long)|丢弃了前 N 个元素的流元素， 与 `limit` 互补|
|takeWhile(Predicate<T>)|（仅限 Java 9）在第一个提供的预期不是 true 的元素处阶段的流元素|
|dropWhile(Predicate<T>)|（仅限 Java 9）丢弃了所提供的预期为 true 的初始元素分段的流元素|

## 终止操作
结束流

|操作|描述|
| ------ | ------------------------ |
|forEach(Consumer<T> action)|将提供的操作应用于流的每个元素。|
|toArray()|使用流的元素创建一个数组。|
|reduce(...)|将流的元素聚合为一个汇总值。|
|collect(...)|将流的元素聚合到一个汇总结果容器中。|
|min(Comparator<T>)|通过比较符返回流的最小元素。|
|max(Comparator<T>)|通过比较符返回流的最大元素。|
|count()|返回流的大小。|
|{any,all,none}Match(Predicate<T>)|返回流的任何/所有元素是否与提供的预期相匹配。|
|findFirst()|返回流的第一个元素（如果有）。|
|findAny()|返回流的任何元素（如果有）。|

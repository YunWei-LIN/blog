---
title: Java IO
date: 2017-06-27 19:08:03
tags: [java, IO, NIO, NIO2]
categories: java
---

___主要内容___

I/O 就是数据的输入/输出。 Java 平台提供了丰富的类库来满足可能的I/O操作需求。
最初的java.io包， JDK 1.4 的 NIO， JDK 7 的 NIO.2。
最早的java.io包把IO操作抽象成数据的流动，进而有了流(Stream)概念；
Java NIO中把IO操作抽象成端到端的一个数据连接，进而有了通道(channel)概念。
如果需要开发高性能网络应用， Java提供的标准库所支持的抽象层次过低， 推荐Netty。

JDK 7 后都建议用 try-with-resources来使用流和通道。

<!-- more -->

## 流
java中 最基本的流就是在字节这个层次上处理的。

### 基本输入流 `InputStream`
#### 数据功能
`read`方法
+ 每次读一个字节
+ 字节数组缓冲区 
最常用， 循环读取， 直到返回-1或异常
+ 字节数组缓冲区和起始位置和长度

#### 控制功能
+ close 关闭
+ skip 跳过指定数目的字节
+ mark 和 reset 标记和重置配合，实现流中部分内容的重复读取
不是所有流都支持mark， 需要用 `markSupperted()`检查
+ available 非阻塞的获取可供读取的字节数


### 基本输出流 `OutputStream`
#### 数据功能
`write` 方法， 类似 `read` 一样， 3种重载形式：每次写入一个字节, 也可写入一个字节数组的全部或部分内容。

#### 控制功能
+ close 关闭
+ flush 强制保存在缓冲区的内容立即进行实际的写入操作。

### 输入流复用
两种方式：
+ 利用标记和重置
使用`BufferedInputStream`
+ 输入流转换成数据
直接把流中所有数据读取到一个字节数组中

这两种复用流的思路一致， 就是预先把需要复用的数据保存起来。

### 过滤输入输出流
+ 缓冲区
BufferedInputStream， BufferedOutputStream
+ java基本类型支持
DataInputStream， DataOutputStream
+ java 基本类型和对象支持
ObjectInputStream， ObjectOutputStream
+ 数据回流
PushbackInputStream， PushbackOutputStream
+ 文件流
FileInputStream， FileOutputStream
+ 管道流
PipedInputStream， PipedOutputStream
+ 顺序连接
SequenceInputStream， SequenceOutputStream

### 字符流
人看字节

## 缓冲区

## 通道

## NIO.2

 

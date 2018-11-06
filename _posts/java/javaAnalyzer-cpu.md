---
title: JVM线上调优(二)  CPU调优
date: 2018-11-07 21:31:21
tags: [java, CPU调优, jstack]
categories: java
---

主要内容

JVM调优的工具和方法 深入浅出，分如下3节介绍，可以解决实际问题。
本章解决java线上CPU调优

+ [工具介绍](/2018/11/06/java/javaAnalyzer/)
公欲善其事，必先利其器
jps, jstat, jmap, jhat, jstack, jinfo, JConsole, VisualVM, Eclipse Memory Analyzer(MAT)
  
+ [CPU调优](/2018/11/07/java/javaAnalyzer-cpu/)
还我CPU
  
+ [Memory调优](/2018/11/07/java/javaAnalyzer-memory/)
吃我的吐出来
  





*更新历史*


<!-- more -->
---
## 定位线程
+ TOP命令查看到CPU的占用情况
TOP --> P : 按CPU使用率排序
![](/images/jvm/ja_cpu_p.png)

+ 具体线程
Java是一个多线程应用，进程是由多个线程构成的，上面看到的是这个进程的CPU占用率，导致这个进程CPU偏高的是其中某个或某几个线程，因而我们需要找到这些线程。
ps命令查看指定进程的线程情况
    ```sh
ps -mp <pid> -o THREAD,tid,time
```
![](/images/jvm/ja_cpu_thread.png)
线程tid为 9100 的线程，CPU占用率达到了99.8%， 就是这个线程的问题。

## 具体分析
请出 `jstack` 分析具体线程。

** 注意： 进制转换 **
ps命令查看到的线程ID 9100 是十进制，`jstack` 命令输出的线程ID可能是十六进制，使用如下命令转换
```sh
╰─$ printf "%x\n" 9100
238c
```

然后使用 `jstack` 定位具体的问题

```sh
╰─$ jstack 9078|grep 238c -A 15
"Thread-0" #10 prio=5 os_prio=0 tid=0x00007fbea0821800 nid=0x238c runnable [0x00007fbe8bbfa000]
   java.lang.Thread.State: RUNNABLE
        at com.xxx.xxx.MyClassLoader$1.run(MyClassLoader.java:19)

...
```
![](/images/jvm/ja_cpu_source.png)

最后结合源码检查就ok。

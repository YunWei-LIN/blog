---
title: JVM线上调优(一) 工具介绍 
date: 2018-11-06 21:31:21
tags: [java, jvm, jps, jstat, jmap, jhat, jstack, jinfo, JConsole, VisualVM, MAT]
categories: java
---

主要内容

JVM调优的工具和方法 深入浅出，分如下3节介绍，可以解决实际问题。

本章首先介绍各种倚天剑和屠龙刀
（致敬 金庸大大）

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

## jps
JVM Process Status Tool,显示指定系统内所有的HotSpot虚拟机进程。

### 命令
    jps [-q] [-mlvV] [<hostid>]
参数都非必须
    
### 参数
    -q : 只输出LVMID
    -l : 输出主类全名或jar路径
    -m : 输出JVM启动时传递给main()的参数
    -v : 输出JVM启动时显示指定的JVM参数

### 例
    ╰─$ jps -l   
    15346 org.jetbrains.idea.maven.server.RemoteMavenServer
    15527 org.jetbrains.jps.cmdline.Launcher
    14986 com.intellij.idea.Main
    30074 jdk.jcmd/sun.tools.jps.Jps
    18941 org.jetbrains.jps.cmdline.Launcher

## jstat 性能分析
jstat(JVM statistics Monitoring)是用于监视虚拟机运行时状态信息的命令，类装载、内存、垃圾收集、JIT编译等数据

### 命令
    jstat -<option> [-t] [-h<lines>] <vmid> [<interval> [<count>]]

### 参数
    [option] : 操作参数
    vmid : 本地虚拟机进程ID
    [interval] : 连续输出的时间间隔
    [count] : 连续输出的次数

#### 参数一览
<style>
table th:first-of-type {
    width: 20%;
}
</style>

|Option|解释|
|---|---|
|gc|垃圾回收堆的行为统计。Statistics of the behavior of the garbage collected heap.|
|class|class loader的行为统计。Statistics on the behavior of the class loader.|
|compiler|HotSpt JIT编译器行为统计。Statistics of the behavior of the HotSpot Just-in-Time compiler.|
|gccapacity|各个垃圾回收代容量(young,old,perm)和他们相应的空间统计。Statistics of the capacities of the generations and their corresponding spaces.|
|gcutil|垃圾回收统计概述。Summary of garbage collection statistics.|
|gccause|垃圾收集统计概述（同-gcutil），附加最近两次垃圾回收事件的原因。Summary of garbage collection statistics (same as -gcutil), with the cause of the last and|
|gcnew|新生代行为统计。Statistics of the behavior of the new generation.|
|gcnewcapacity|新生代与其相应的内存空间的统计。Statistics of the sizes of the new generations and its corresponding spaces.|
|gcold|年老代和永生代行为统计。Statistics of the behavior of the old and permanent generations.|
|gcoldcapacity|年老代行为统计。Statistics of the sizes of the old generation.|
|gcpermcapacity|永生代行为统计。Statistics of the sizes of the permanent generation.|
|printcompilation|HotSpot编译方法统计。HotSpot compilation method statistics.|

`-gc`
常用命令 垃圾回收堆的行为统计

    ╰─# jstat -gc 15490 3s
    S0C    S1C    S0U    S1U      EC       EU        OC         OU       MC     MU    CCSC   CCSU   YGC     YGCT    FGC    FGCT     GCT   
    8192.0 33792.0 8181.8  0.0   724992.0 592657.6  215040.0   81474.7   90880.0 86753.7 11520.0 10764.0     20    0.625   3      0.365    0.990
    8192.0 33792.0 8181.8  0.0   724992.0 594107.5  215040.0   81474.7   90880.0 86753.7 11520.0 10764.0     20    0.625   3      0.365    0.990
    8192.0 33792.0 8181.8  0.0   724992.0 594455.2  215040.0   81474.7   90880.0 86753.7 11520.0 10764.0     20    0.625   3      0.365    0.990
    8192.0 33792.0 8181.8  0.0   724992.0 595905.2  215040.0   81474.7   90880.0 86753.7 11520.0 10764.0     20    0.625   3      0.365    0.990

**C即Capacity 总容量，U即Used 已使用的容量**

+ S0C：第一个幸存区的大小
+ S1C：第二个幸存区的大小
+ S0U：第一个幸存区的使用大小
+ S1U：第二个幸存区的使用大小
+ EC：伊甸园区的大小
+ EU：伊甸园区的使用大小
+ OC：老年代大小
+ OU：老年代使用大小
+ MC：方法区大小
+ MU：方法区使用大小
+ CCSC:压缩类空间大小
+ CCSU:压缩类空间使用大小
+ YGC：年轻代垃圾回收次数
+ YGCT：年轻代垃圾回收消耗时间
+ FGC：老年代垃圾回收次数
+ FGCT：老年代垃圾回收消耗时间
+ GCT：垃圾回收消耗总时间

更多可参考[jstat](https://app.yinxiang.com/shard/s70/nl/17973191/9a8fdbd4-6a6b-46fb-8c70-babd70a02fcb) [java高分局之jstat命令使用](https://blog.csdn.net/maosijunzi/article/details/46049117)


## jinfo
jinfo(JVM Configuration info)实时查看和调整虚拟机运行参数

### 命令
    jinfo <option> <pid>
    
### option
    -flag : 输出指定args参数的值
    -flags : 不需要args参数，输出所有JVM参数的值
    -sysprops : 输出系统属性，等同于System.getProperties()

## jmap 查看内存
jmap(JVM Memory Map)命令用于生成heap dump文件，如果不使用这个命令，还可以使用-XX:+HeapDumpOnOutOfMemoryError参数来让虚拟机出现OOM的时候自动生成dump文件。
又或者在Linux系统下通过Kill-3命令发送进程退出信号“吓唬”一下虚拟机，也能拿到dump文件。
jmap不仅能生成dump文件，还可以查询finalize执行队列、Java堆和永久代的详细信息，如当前使用率、当前使用的是哪种收集器等。

### 安装
openjdk 需要 安装 `debuginfo`，版本需要和openjdk 一致

```sh
yum update java-1.8.0-openjdk.x86_64
yum --enablerepo="*-debug*" install java-1.8.0-openjdk-debuginfo
```

否则 异常
```
- unknown CollectedHeap type : class sun.jvm.hotspot.gc_interface.CollectedHeap
```

### 命令
    jmap [option] pid

### option参数
    dump : 生成堆转储快照
    finalizerinfo : 显示在F-Queue队列等待Finalizer线程执行finalizer方法的对象
    heap : 显示Java堆详细信息
    histo : 显示堆中对象的统计信息
    clstats : 类加载的统计信息
    F : 当-dump没有响应时，强制生成dump快照
    
### 例
+ -dump
        jmap -dump:live,format=b,file=dump.hprof $pid

dump.hprof这个后缀是为了后续可以直接用MAT(Memory Anlysis Tool)打开。

+ -finalizerinfo
打印等待回收对象的信息

+ -heap
打印heap的概要信息，GC使用的算法，heap的配置及wise heap的使用情况,可以用此来判断内存目前的使用情况以及垃圾回收情况

    ```sh
    ╰─# jmap -heap 15490
    Attaching to process ID 15490, please wait...
    Debugger attached successfully.
    Server compiler detected.
    JVM version is 25.191-b12

    using thread-local object allocation.
    Parallel GC with 2 thread(s) //GC 方式  

    Heap Configuration:
        MinHeapFreeRatio         = 0 //对应jvm启动参数-XX:MinHeapFreeRatio设置JVM堆最小空闲比率(default 40)
        MaxHeapFreeRatio         = 100 //对应jvm启动参数 -XX:MaxHeapFreeRatio设置JVM堆最大空闲比率(default 70)
        MaxHeapSize              = 4164943872 (3972.0MB) //对应jvm启动参数-XX:MaxHeapSize=设置JVM堆的最大大小
        NewSize                  = 87031808 (83.0MB) //对应jvm启动参数-XX:NewSize=设置JVM堆的‘新生代’的默认大小
        MaxNewSize               = 1388314624 (1324.0MB) //对应jvm启动参数-XX:MaxNewSize=设置JVM堆的‘新生代’的最大大小
        OldSize                  = 175112192 (167.0MB) //对应jvm启动参数-XX:OldSize=<value>:设置JVM堆的‘老生代’的大小
        NewRatio                 = 2 //对应jvm启动参数-XX:NewRatio=:‘新生代’和‘老生代’的大小比率
        SurvivorRatio            = 8 //对应jvm启动参数-XX:SurvivorRatio=设置年轻代中Eden区与Survivor区的大小比值 
        MetaspaceSize            = 21807104 (20.796875MB) //对应jvm启动参数-XX:MetaspaceSize=<value>:设置JVM堆的‘方法区’的初始大小
        CompressedClassSpaceSize = 1073741824 (1024.0MB)
        MaxMetaspaceSize         = 17592186044415 MB //对应jvm启动参数-XX:MaxMetaspaceSize=<value>:设置JVM堆的‘方法区’的最大大小
        G1HeapRegionSize         = 0 (0.0MB)

    Heap Usage: //堆内存使用情况
    PS Young Generation
    Eden Space: //Eden区内存分布
        capacity = 742391808 (708.0MB) //Eden区总容量
        used     = 626604800 (597.576904296875MB) //Eden区已使用
        free     = 115787008 (110.423095703125MB) //Eden区剩余容量
        84.40351755605579% used //Eden区使用比率
    From Space: //其中一个Survivor区的内存分布
        capacity = 8388608 (8.0MB)
        used     = 8378128 (7.9900054931640625MB)
        free     = 10480 (0.0099945068359375MB)
        99.87506866455078% used
    To Space: //另一个Survivor区的内存分布
        capacity = 34603008 (33.0MB)
        used     = 0 (0.0MB)
        free     = 34603008 (33.0MB)
        0.0% used
    PS Old Generation //当前的Old区内存分布
        capacity = 220200960 (210.0MB)
        used     = 83430072 (79.56511688232422MB)
        free     = 136770888 (130.43488311767578MB)
        37.88815089634487% used

    38773 interned Strings occupying 4270344 bytes.

    ```
+ -histo
打印堆的对象统计，包括对象数、内存大小等等 （因为在dump:live前会进行full gc，如果带上live则只统计活对象，因此不加live的堆大小要大于加live堆的大小 ）

    ```sh
$ jmap -histo:live 15490 | more

 num     #instances         #bytes  class name
----------------------------------------------
   1:        137190       15505768  [C
   2:         10772        8598936  [B
   3:        131166        3147984  java.lang.String
   4:         29783        2620904  java.lang.reflect.Method
   5:         36792        2354688  com.mysql.jdbc.ConnectionPropertiesImpl$BooleanConnectionProperty
   6:         72596        2323072  java.util.concurrent.ConcurrentHashMap$Node
   7:         17598        1948272  java.lang.Class
   8:         14682        1766000  [Ljava.util.HashMap$Node;
   9:         55163        1765216  java.util.Hashtable$Entry
  10:         13793        1732072  [I
  ....
```

    xml class name是对象类型，说明如下：

        B  byte
        C  char
        D  double
        F  float
        I  int
        J  long
        Z  boolean
        [  数组，如[I表示int[]
        [L+类名 其他对象


+ -clstats
打印Java堆内存的类加载器的统计信息。对于每个类加载器而言，它的名称、活跃度、地址、父类加载器、它所加载的类的数量和大小都会被打印。此外，包含的字符串数量和大小也会被打印。

    ```sh
jmap -clstats 15490

```

## jhat
jhat(JVM Heap Analysis Tool)命令是与jmap搭配使用，用来分析jmap生成的dump。
千万下载dump文件到开发环境进行分析。
推荐使用 Eclipse Memory Analyzer(MAT)

## jstack 查看线程
jstack（Stack Trace for Java）用于生成虚拟机当前时刻的线程快照（一般称为threaddump或者javacore文件）。
线程快照是当前java虚拟机内每一条线程正在执行的方法堆栈的集合，生成线程快照的主要目的是**定位线程出现长时间停顿**的原因，如**线程间死锁、死循环、请求外部资源**等。
如果现在运行的java程序呈现停顿状态，jstack是非常有用的。

### 命令
    jstack [-l][-e] <pid>
    
### Options
    -l  long listing. Prints additional information about locks
    -e  extended listing. Prints additional information about threads
    
### 解析
参考 [jstack](http://www.hollischuang.com/archives/110)


---

以下都是可视化的工具
## JConsole
JConsole（Java Monitoring and Management Console）是一种基于JMX的可视化监视、 管理工具

## VisualVM
大名鼎鼎的 VisualVM， 基于NetBeans平台开发，通过插件扩展支持，VisualVM可以做到
+ 显示虚拟机进程以及进程的配置、 环境信息（jps、 jinfo）。
+ 监视应用程序的CPU、 GC、 堆、 方法区以及线程的信息（jstat、 jstack）。
+ dump以及分析堆转储快照（jmap、 jhat）。
+ 方法级的程序运行性能分析，找出被调用最多、 运行时间最长的方法。
+ 离线程序快照：收集程序的运行时配置、线程dump、内存dump等信息建立一个快照，可以将快照发送开发者处进行Bug反馈。

![](/images/jvm/ja_visualvm.jpg)

## [Eclipse Memory Analyzer(MAT)](https://www.eclipse.org/mat/)
Eclipse 出品，MAT可以对堆dump的文件进行分析，可以去detail页看线程各个对象的使用数目等情况。

+ 内存泄露报表，自动检查可能存在内存泄露的对象，通过报表展示存活的对象以及为什么他们没有被垃圾收集；
+ 对象报表，对可颖对象的分析，如字符串是否定义重了，空的collection、finalizer以及弱引用等。

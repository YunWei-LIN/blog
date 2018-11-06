---
title: JVM线上调优(三)  内存调优
date: 2018-11-07 21:31:21
tags: [java, 内存泄漏, 内存调优]
categories: java
---

主要内容

JVM调优的工具和方法 深入浅出，分如下3节介绍，可以解决实际问题。

本章讨论java线上内存调优

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
Java应用占用太多内存也有可能的确是内存硬件不足或JVM设置的太小，记得最后考虑下。

本文主要讨论的是Java内存泄漏。

## 泄漏特征
出现如下情况之一，就需要检查了
+ OOM Out-of-Memory
+ 堆(老年代)/方法区 不停地增长，频繁FullGC
+ 不明的crash
+ 数据集越大，性能越低


## 泄漏原因
+ File/Text buffers 等资源没关闭

+ 静态集合类引用
静态变量的生命周期和应用程序一致，他们所引用的所有的对象也不能被释放。

+ 监听器
只增加监听器，不删除

+ 各种连接没有释放

+ 单例模式
单例对象在被初始化后将在JVM的整个生命周期中存在，单例对象持有外部对象的引用，那么这个外部对象将不能被jvm正常回收

+ 内部类和外部模块等的引用
非静态内部类的对象会隐式强引用其外围对象，所以在内部类未释放时，外围对象也不会被释放，从而造成内存泄漏。

+ 集合中的可变对象修改
一般是HashSet, HashMap, 主键的key的hashCode变化以后，添加或者删除都是映射到不同的桶中。所以对于HashSet或者HashMap的Key，都应该是不可变类型。
```java
class Person{
    private String name;
    private int age;

    // ** 改变了 key **
    @Override
    public int hashCode() {
        return name.hashCode() + age;
    }

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
...

    public void setAge(int age) {
        this.age = age;
    }
}


    public static void main(String[] args) {
        Set<Person> persons = new HashSet<>();
        Person p1 = new Person("AAAA",  1);
        Person p2 = new Person("BBBB",  2);
        Person p3 = new Person("CCCC", 3);
        persons.add(p1);
        persons.add(p2);
        persons.add(p3);
        System.out.println("size:" + persons.size()); // 3

        p3.setAge(31); // 修改属性。
        persons.remove(p3); // 移除不掉.
        persons.add(p3); // 添加成功.

        System.out.println("size:" + persons.size()); // 4
    }
```

## 解决
### OOM
java启动加入`-XX:+HeapDumpOnOutOfMemoryError`，发生OOM时自动生成dump文件
    java ... -XX:+HeapDumpOnOutOfMemoryError  -XX:HeapDumpPath=./oom.hprof
    
发生OOM异常时需要对该文件进行分析。把oom.hprof 下载到开发环境，使用 `MAT` 分析。

### 其他
+ `jps` 找到java进程ID[pid]

+ top -p [pid] 查看内存使用情况
```sh
╰─$ top -p 14986
top - 17:27:51 up 29 days,  7:59,  6 users,  load average: 1.27, 0.76, 0.77
Tasks:   1 total,   0 running,   1 sleeping,   0 stopped,   0 zombie
%Cpu(s):  8.8 us,  7.3 sy,  0.0 ni, 83.4 id,  0.0 wa,  0.0 hi,  0.5 si,  0.0 st
KiB Mem : 16287752 total,  5915300 free,  6235960 used,  4136492 buff/cache
KiB Swap: 16784380 total, 16588028 free,   196352 used.  9302212 avail Mem 

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND                                                                                                                                      
14986 sam       20   0 5682796 2.078g  51756 S 2.000 13.38 451:02.04 java                                                                                                                                         
```

+ FullGC情况
`jstat -gcutil [pid] 3s` 每3s的GC情况，主要看 FGC

    ```sh
╰─$ jstat -gcutil 14986 3s
  S0     S1     E      O      M     CCS    YGC     YGCT    FGC    FGCT    CGC    CGCT     GCT   
  0.00  51.09   9.93  61.42  93.53  90.33   2961   45.827   416  304.043     -        -  349.870
  0.00  51.09   9.96  61.42  93.53  90.33   2961   45.827   416  304.043     -        -  349.870
  0.00  51.09  10.18  61.42  93.53  90.33   2961   45.827   416  304.043     -        -  349.870
  0.00  51.09  10.34  61.42  93.53  90.33   2961   45.827   416  304.043     -        -  349.870
  0.00  51.09  11.28  61.42  93.53  90.33   2961   45.827   416  304.043     -        -  349.870
  。。。
```

+ jmap
查看目前的各种类型的对象创建数目和所占用内存

    ```sh
╰─$ jmap -histo:live 14986|more

 num     #instances         #bytes  class name
----------------------------------------------
   1:        514181       53559808  [C
   2:         90742       48738768  [B
   3:         76202       17773512  [I
   4:        152743       13160544  [Ljava.lang.Object;
   5:        509711       12233064  java.lang.String
   6:         83468        9177120  java.lang.Class
   7:        117955        3774560  java.util.HashMap$Node
   8:        100076        3202432  java.util.concurrent.ConcurrentHashMap$Node
   9:         11155        2562688  [J
  10:         15235        2375912  [Ljava.util.HashMap$Node;
  11:         73180        2341760  com.intellij.util.text.ByteArrayCharSequence
。。。。
```
  还可以生成JVM的内存dump文件，下载到本地使用 `MAT` 分析
        jmap -dump:format=b,file=文件名 [pid]
    

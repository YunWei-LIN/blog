---
title: Java 逃逸分析 (Escape Analysis)
date: 2018-09-30 22:18:03
tags: [java, Escape Analysis]
categories: java
---

___主要内容___

逃逸分析的基本行为就是分析对象动态作用域：当一个对象在方法中被定义后，它可能被外部方法所引用，称为方法逃逸。甚至还有可能被外部线程访问到，譬如赋值给类变量或可以在其他线程中访问的实例变量，称为线程逃逸。
即时编译器（Just-in-time Compilation，JIT）判断对象是否逃逸的依据，一是对象是否被存入堆中（静态字段或者堆中对象的实例字段），二是对象是否被传入未知代码中。
逃逸分析 在 方法内联后才进行。

如果对象没有逃逸，即时编译器对代码进行性能优化。

<!-- more -->

## 逃逸方式
show you code
```java
public class Escape {
    public static Object obj;
    public void globalVariableEscape() {  // 给全局变量赋值，发生逃逸
        obj = new Object();
    }
    
    public Object methodEscape() {  // 方法返回值，发生逃逸
        return new Object();
    }
    public void methodNoEscape() {  // 方法返回值，没有逃逸
        Object obj = new Object();
        return null;
    }
    
    
    public void instanceEscape() {  // 实例引用发生逃逸
        test(this); 
    }
}

```

## 基于逃逸分析的优化
一旦对象没有逃逸，那么可能做如下优化： 字段访问优化（锁消除、栈上分配以及标量替换）， 字段存储优化， 死代码消除， 循环优化

### 字段访问优化
+ 锁消除
线程同步本身比较耗，如果确定一个对象不会逃逸出线程，无法被其它线程访问到，那该对象的读写就不会存在竞争，对这个变量的同步措施就可以消除掉。单线程中是没有锁竞争。（锁和锁块内的对象不会逃逸出线程就可以把这个同步块取消）

+ 栈上分配
如果一个对象仅在某线程中被分配，要使指向该对象的指针永远不会逃逸，对象可能是栈分配的候选，而不是堆分配。
一旦对象分配到栈上，方法执行完后自动销毁，而不需要垃圾回收的介入，从而提高系统性能。

+ 标量替换
将原本连续分配的对象拆散为一个个单独的字段，分布在栈上或者寄存器中。我的理解就是 将对象拆解为局部变量。
减少对 对象以及对象内属性 的访问，  因为他们都涉及到访问内存，而局部变量 访问 栈和控制计数器， 不需要垃圾回收的介入。<br>
比如：
```java
static int bar(Foo o, int x) {
    int y = o.a + x;
    return o.a + y;
}

//在上面这段代码中，实例字段将被读取两次。即时编译器会将第一次读取的值缓存起来，并且替换第二次字段读取操作，以节省一次内存访问。
static int bar(Foo o, int x) {
int t = o.a; 
int y = t + x;
return t + y;
}

```
注意 `volatile` 修饰的字段 不会被完全优化

### 字段存储优化
编译器还将消除冗余的存储节点。
如果一个字段先后被存储了两次，而且这两次存储之间没有对第一次存储内容的读取，那么即时编译器可以将第一个字段存储给消除掉。当然，如果所存储的字段被标记为 volatile，那么即时编译器也不能将冗余的存储操作消除掉。这种情况看似很蠢，但实际上并不少见，比如说两个存储之间隔着许多其他代码，或者因为方法内联的缘故，将两个存储操作（如构造器中字段的初始化以及随后的更新）纳入同一个编译单元里。

### 死代码消除
即删除无用字段
死存储还有一种变体，即在部分程序路径上有冗余存储。
另一种死代码消除则是不可达分支消除。不可达分支就是任何程序路径都不可到达的分支，我们之前已经多次接触过了。

### 循环无关代码外提
所谓的循环无关代码（Loop-invariant Code），指的是循环中值不变的表达式。如果能够在不改变程序语义的情况下，将这些循环无关代码提出循环之外，那么程序便可以避免重复执行这些表达式，从而达到性能提升的效果。


## 逃逸分析并不成熟
无法保证逃逸分析的性能消耗一定能高于他的消耗。虽然经过逃逸分析可以做标量替换、栈上分配、和锁消除。但是逃逸分析自身也是需要进行一系列复杂的分析的，这其实也是一个相对耗时的过程。

一个极端的例子，就是经过逃逸分析之后，发现没有一个对象是不逃逸的。那这个逃逸分析的过程就白白浪费掉了。
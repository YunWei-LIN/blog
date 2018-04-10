title: 对象的引用 reference
date: 2015-10-15 15:22:03
tags: [java, reference]
categories: 杂谈
---
  很多现代语言都有 `GC` ，与 `GC` 紧密相关的是对象的引用。

## Java
Java有4种引用类型：
* 强引用 StrongReference： 平常使用的都是这种，其他引用统称 `弱引用` 。

  强引用可以直接访问目标对象。
  强引用所指向的对象在任何时候都不会被系统回收。
  强引用可能导致内存泄漏。
```
Bean bean = new Bean();
```

* 软引用 SoftReference： 最强的弱引用， 内存紧缺时可能会被GC回收。

  软引用使用 get() 方法取得对象的强引用从而访问目标对象。
  软引用所指向的对象按照 JVM 的使用情况（Heap 内存是否临近阈值）来决定是否回收。
  软引用可以避免 Heap 内存不足所导致的异常。
```
SoftReference<Bean> bean = new SoftReference<Bean>(new Bean());
```

* 弱引用 WeakReference ：

  弱引用使用 get() 方法取得对象的强引用从而访问目标对象。
  一旦系统内存回收，无论内存是否紧张，弱引用指向的对象都会被回收。
  弱引用也可以避免 Heap 内存不足所导致的异常。
```
WeakReference<Bean> bean = new WeakReference<Bean>(new Bean());
```

* 虚引用 PhantomReference： 始终是null

```
ReferenceQueue<Bean> refQueue = new ReferenceQueue<Bean>();
PhantomReference<Bean> referent = new PhantomReference<Bean>(new Bean(), refQueue);
```

有篇可以参照 [blog](http://blog.csdn.net/liuj2511981/article/details/46372187)

## Swift
Swift 使用自动引用计数（ARC）机制来跟踪和管理你的应用程序的内存。
Swift有3种引用类型：
* 强引用 StrongReference： ARC + 1，ARC 不会销毁被引用的实例

```swift
class Person {
  var apartment: Apartment?
}
```

* 弱引用 WeakReference ： 弱引用不会对其引用的实例保持强引用，因而不会阻止 ARC 销毁被引用的实例。

```
class Apartment {
    weak var tenant: Person?
}
```

* 无主引用 unowned ： 无主引用是永远有值的。总是被定义为非可选类型（non-optional type）。总是可以被直接访问。
```
class Customer {
    var card: CreditCard?
}
class CreditCard {
    unowned let customer: Customer
}

```

* 无主引用以及隐式解析可选属性
```
class Country {
    var capitalCity: City!
}

class City {
    unowned let country: Country
}
```


Person和Apartment的例子展示了两个属性的值都允许为nil，并会潜在的产生循环强引用。这种场景最适合用弱引用来解决。

Customer和CreditCard的例子展示了一个属性的值允许为nil，而另一个属性的值不允许为nil，这也可能会产生循环强引用。这种场景最适合通过无主引用来解决。

Country和City的例子中两个属性都必须有值，并且初始化完成后永远不会为nil。在这种场景中，需要一个类使用无主属性，而另外一个类使用隐式解析可选属性。


更详细的内容可以参考 [中译 Swift](http://wiki.jikexueyuan.com/project/swift/chapter2/16_Automatic_Reference_Counting.html#7be938fb77dc56602eecdece3e1b5847)

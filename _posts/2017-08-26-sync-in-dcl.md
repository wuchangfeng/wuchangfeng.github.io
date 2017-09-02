---
layout: post
title: 深入理解 Java 之 Sync 到底锁住了啥
date: 2017-08-26 20:48:13 +0800
categories: 
---

### 问题描述

一个面试题，觉得很有意思，先看下面这个 DCL 单利模式：

```java
public class Singleton {
    private volatile static Singleton instance; //声明成 volatile
    private Singleton (){}
    public static Singleton getSingleton() {
        if (instance == null) {                         
            synchronized (Singleton.class) {
                if (instance == null) {       
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
   
}
```

这个单利模式能写出来也算可以了，但是问题在于里面涉及的内容就很多：

1：为什么构造函数要用private修饰？

2：为什么要两次判断instance是否为null？

3：为什么要用synchronized来修饰，与lock有什么区别？

4：sync中的Singleton.class是什么意思？为什么不是this？

5：volatile有什么作用？放在这个单利中有什么作用？

这些问题都非常有意思，可以学习的知识点超级多，有不太明白的可以去Google一下，今天我们关注点是第4个问题，为什么是Singleton.class，是什么意思呢？放在括号里的作用是什么？

### 问题发散

我们知道syhchronized(后面用sync替代)是用来控制线程同步的，其可以用在代码块上也可以用在方法上。但是sync很"难"用，且看下段代码,首先说明一下这段代码的作用，用sync来锁住一个方法，让不同线程能够顺序执行这段方法，不理解没关系，后面看结果就行：

```java
public class SyncDemo {

    static class Sync{
        public synchronized void method(){
            System.out.println("method begin");
            try {
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("method end");
        }
    }


    static class MyThread extends Thread{
        @Override
        public void run() {
            Sync sync = new Sync();
            sync.method();
        }
    }


    public static void main(String[] args) {
        for (int i = 0; i < 3; i++) {
            Thread thread = new MyThread();
            thread.start();
        }
    }
}

```

打印出来：

```java
method begin
method begin
method begin
method end
method end
method end
```

看结果吧，没有达到我们预期的目的（一次一个begin和一个method一起执行）。呀，为啥呢。既然锁不住方法，我们就去锁一下代码块好了，去掉method方法中修饰的sync，在method方法中用sync来修饰代码块：

```java
static class Sync{
        public void method() {
            synchronized (this) {
                System.out.println("method begin");
                try {
                    Thread.sleep(3000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("method end");
            }
        }
  }
```

我们打印结果，发现仍然没有变化，sync还是没有起作用...

```java
method begin
method begin
method begin
method end
method end
method end
```

实际上，sync(this)以及非static的sync方法，只能防止多个线程同时执行同一个对象的同步代码段。我们回归原来的问题，sync到底锁的是对象还是代码段？答案是：sync锁住的是括号里面的对象，而不是代码段。对于非static的sync方法，锁住的就是对象本身，也就是this。

### 验证结论

为了验证上述结论，我们修改一下代码，让三个线程执行的对象是同一个Sync对象：

```java
 static class MyThread extends Thread{
        private Sync sync;

        public MyThread(Sync sync) {
            this.sync = sync;
        }

        @Override
        public void run() {
            sync.method();
        }
    }


    public static void main(String[] args) {
       // 唯一性
        Sync sync = new Sync();
        for (int i = 0; i < 3; i++) {
            Thread thread = new MyThread(sync);
            thread.start();
        }
    }
```

这次打印出的结果如下所示：

```
method begin
method end
method begin
method end
method begin
method end
```

可以看到达到目的啦，等等，达到什么目的？我们的目的不就是想让每个线程去单独使用Sync类的对象中的method方法吗？但是，如果我们还是想去锁住那一段代码，就是最开始的示例中的method方法，有什么解决办法？其实解决办法是有的，就是让sync去锁住同一个对象就好了，同行我们的做法是用sync去锁住这个类的class对象，即`sync(Sync.class)`,换一种说法就是用`sync(Sync.class)`实现了全局锁的效果。另外，用static修饰的method中当然无法使用this啦，不可避免的要去锁住类的Class对象，好像跟前面的DCL也有那么点关系哟。

### 深入进去

先上结论：Java中的每一个对象都可以作为锁，这是sync实现同步的基础：

1. 普通同步方法，锁是当前实例对象 
2. 静态同步方法，锁是当前类的class对象 
3. 同步方法块，锁是括号里面的对象

写一个Demo类如下所示：

```java
public class SyncDemo01 {
    public synchronized void method1(){

    }

    public void method2(){
        synchronized (this){

        }
    }
}
```

然后我们用javap工具去分析一下这个类的Class文件，第一步先用javac编译出Class文件：

```
javac SyncDemo01.java
```

然后用javap工具查看Class文件：

```
javap -c SyncDemo01 
```

具体Class文件格式如下所示，我在里面加入了注释，方便查看：

```c
Compiled from "SyncDemo01.java"
public class SyncDemo01 {
  public SyncDemo01();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: return

  // method1 对应
  public synchronized void method1();
    Code:
       0: return

 // method2 对应
  public void method2();
    Code:
       0: aload_0
       1: dup
       2: astore_1
       3: monitorenter // 注释1
       4: aload_1
       5: monitorexit // 注释2
       6: goto          14
       9: astore_2
      10: aload_1
      11: monitorexit
      12: aload_2
      13: athrow
      14: return
    Exception table:
       from    to  target type
           4     6     9   any
           9    12     9   any
}

```

从上面反编译出来的Class文件看出method2(同步代码块)，指令较多比较好分析，method1(同步方法)指令太少，暂时还无法分析。根据相关博客与书籍，先总结如下：

- 同步代码块：monitorenter指令插入到同步代码块开始的位置，monitorexit指令插入到同步代码块结束的位置，enter和exit一一对应。任何一个对象都有一个monior与之对应，线程执行到monitorenter指令时，表示该对象的monitor所有权将被持有，即被锁定。
- 同步方法：如上Class文件，在JVM层面没有任何特别的指令来实现被sync修饰的方法，而是在Class文件的**方法表**中将该方法的access_flag字段中的sync标志位置置为1，表示该方法是同步方法并使用调用该方法的对象或者该方法所属的Class在JVM的内部对象表示Klass作为锁对象。

### 继续深入

前面我们提到了monitor的概念，其实sync得以实现的重要基础。下面将要去了解对象头和Monitor的概念。

为什么先要去了解对象头呢，因为sync用的锁是存在Java对象头中的。Hotspot虚拟机对象头主要包含：MarkWord、KlassPointer 分别是标记字段和类型指针。其中KP是对象指向它的类元数据的指针，虚拟机通过这个指针来确定对像那个是哪个类的实例，MW用于存储对象自身的运行时数据，它是实现轻量级和偏向锁的关键。

MW用于存储对象自身的运行时数据，如哈希码、GC分代年龄、锁状态标志、线程持有的锁、偏向线程ID和偏向时间戳等。

Monitor可以理解成一个同步工具，也可以是一个同步机制。所有的Java对象都是天生的Monitor，每一个对象都可以成为Monitor的潜质。其实线程私有的数据结构，每一个线程都有一个可用的monitor record列表，同时还有一个全局的可用列表。每一个被锁住的对象都会和一个monitor关联。



哎，我曹，写不动了。。。关于各种sync锁详细知识可以参见[死磕Java并发值sync关键字](http://blog.csdn.net/chenssy/article/details/54883355)


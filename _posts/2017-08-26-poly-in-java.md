---
layout: post
title: 深入理解 Java 之多态底层实现原理
date: 2017-08-26 21:00:26 +0800
categories: 
---

在理解多态实现机制之前，让我们先弄清楚Java虚拟机的知识，这些基础知识都很重要。我大可用一个Demo来引入话题，但是这样理解起来就不是很透彻。再次向你推荐《深入理解JVM虚拟机》这本书。

### 方法调用与解析

最开始我知道一个说法是这样说的：JVM虚拟机类加载机制中：加载、验证、准备、初始化和卸载这5个阶段的顺序是确定的，然而**解析**阶段则不一定。某些情况解析可以在初始话之后再开始，这样做的目的是为了Java支持运行时绑定(动态绑定或者晚期绑定)的做法。对于这种特殊情况，肯定要去了解一下解析到底是干嘛的了：解析阶段是虚拟机将常量池内的**符号引用**转换为**直接引用**的过程，这个可以展开来说很久，也可以很简单的理解一下：符号引用于虚拟机的内存无关，类似于逻辑地址一样，只是一个符号，最终还是要转换成物理地址也就是直接引用来访问某个目标的。先不要管太多，就知道这个概念给Java带来了强大的扩展能力，方法调用也变得相对复杂起来。在类的加载阶段，会将其中一部分的符号引用转化为直接引用(要求：程序在真正执行之前就有一个可确定的执行版本)，并且这个方法的调用版本在运行期间是不可变的。

Jvm提供了5条方法调用字节码指令，分别如下所示，前面两种方法都可以在类解析阶段确定唯一的调用版本，这些方法称为非虚方法，与之对应的就是虚方法：

* invokestatic：调用静态方法
* invokespecial：调用实例构造器<init>方法，私有方法和父类方法
* invokevirtual：调用所有的虚方法
* invokeinterface：调用接口方法，会在运行时再确定一个实现此接口的对象

**解析调用一定是静态过程，在编译期间就完全确定的。而分派调用则可能是静态的也可能是动态的。**这些概念将在后文有较详细的解释。

### 静态分派

这个概念不看Demo的话有点难以理解，简单理解可以认为是 **Java中重载的体现**，所以话不多说，先看下面一段代码：

```java
public class StaticDispatch {

    static abstract class Human{

    }

    static class Man extends Human{

    }

    static class Woman extends Human{

    }

    public void sayHello(Human guy){
        System.out.println("Hello,guy");
    }

    public void sayHello(Man guy){
        System.out.println("Hello,gentleman");
    }

    public void sayHello(Woman guy){
        System.out.println("Hello,lady");
    }

    public static void main(String[] args) {
        Human man = new Man();// 注释1
        Human woman = new Woman();
        StaticDispatch sd = new StaticDispatch();
        sd.sayHello(man);
        sd.sayHello(woman);
    }
}
```

运行的结果都是：hello,guy ,但是程序为什么选择执行参数类型为Human的重载呢，解决这个问题之前需要先了解下面这两个概念：在上面代码注释1处，把**Human称为静态类型**(外观类型)，**Man称为实际类型**，静态类型编译期可知，而实际类型却只有在运行时才能知道。编译器在编译程序的时候并不知道一个变量的实际类型是什么，类似下面的代码：

```
// 实际类型变化
Human man = new Human();
man = new Woman();
// 静态类型变化
sr.sayHello((Man)man)
sr.sayHello((Woman)man)
```

回到上述代码中，main里面的两次sayHello方法调用，都是在方法接受者是对象“sd”的情况下，使用哪个重载版本完全取决于传入**参数的数量**和**数据类型**。实例代码中刻意的定义了两个静态类型相同但是实际类型不同的变量，但是JVM在重载时是通过参数的**静态类型**而不是**实际类型**作为判定依据的。同时静态类型是编译期可知的，因此在编译阶段，Javac编译器会根据参数的类型来决定选择哪个重载版本。引申出：**所有依赖静态类型来定位方法执行版本的分派动作称为静态分派，其典型应用是方法重载。因此确定静态分派的动作实际上不是由虚拟机来执行的。**另外，编译器虽然能区分出方法的重载版本，但是很多情况具体是哪个版本并不是非0即1的情况，可能会存在一个更合适的情况。这点就不引申了。

### 动态分派

动态分派是多态的另一种体现：重写(override),具体看下面的Demo：

```java
public class DynamicDispatch {
    static abstract class Human{
        protected abstract void sayHello();
    }

    static class Man extends Human {
        @Override
        protected void sayHello() {
            System.out.println("Man say Hello");
        }
    }

    static class Woman extends Human {
        @Override
        protected void sayHello() {
            System.out.println("WoMan say Hello");
        }
    }

    public static void main(String[] args) {
        
      
        man.sayHello();
        woman.sayHello();
        man = new Woman();
        man.sayHello();
    }
}
```

运行结果如下所示：

```java
Man say Hello
WoMan say Hello
WoMan say Hello
```

这个符合面向对象的特点吧，但是我们也要去分析一下：显然这里不可能在根据静态类型来决定，因为静态类型都是Human，导致上述结果明显的原因是两个变量的**实际类型**不同，Java虚拟机是如何根据实际类型来分派方法执行的版本的呢？使用javap命令输出Class文件的字节码看看究竟：

```java
Classfile /Users/allenwu/IdeaProjects/JavaKnow/src/DynamicDispatch.class
  Last modified 2017-8-27; size 514 bytes
  MD5 checksum b7efe1364a3e0b814d98301263382e46
  Compiled from "DynamicDispatch.java"
public class DynamicDispatch
  minor version: 0
  major version: 52
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #8.#22         // java/lang/Object."<init>":()V
   #2 = Class              #23            // DynamicDispatch$Man
   #3 = Methodref          #2.#22         // DynamicDispatch$Man."<init>":()V
   #4 = Class              #24            // DynamicDispatch$Woman
   #5 = Methodref          #4.#22         // DynamicDispatch$Woman."<init>":()V
   #6 = Methodref          #12.#25        // DynamicDispatch$Human.sayHello:()V
   #7 = Class              #26            // DynamicDispatch
   #8 = Class              #27            // java/lang/Object
   #9 = Utf8               Woman
  #10 = Utf8               InnerClasses
  #11 = Utf8               Man
  #12 = Class              #28            // DynamicDispatch$Human
  #13 = Utf8               Human
  #14 = Utf8               <init>
  #15 = Utf8               ()V
  #16 = Utf8               Code
  #17 = Utf8               LineNumberTable
  #18 = Utf8               main
  #19 = Utf8               ([Ljava/lang/String;)V
  #20 = Utf8               SourceFile
  #21 = Utf8               DynamicDispatch.java
  #22 = NameAndType        #14:#15        // "<init>":()V
  #23 = Utf8               DynamicDispatch$Man
  #24 = Utf8               DynamicDispatch$Woman
  #25 = NameAndType        #29:#15        // sayHello:()V
  #26 = Utf8               DynamicDispatch
  #27 = Utf8               java/lang/Object
  #28 = Utf8               DynamicDispatch$Human
  #29 = Utf8               sayHello
{
  public DynamicDispatch();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 8: 0

  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=3, args_size=1
         0: new           #2                  // class DynamicDispatch$Man
         3: dup
         4: invokespecial #3                  // Method DynamicDispatch$Man."<init>":()V
         7: astore_1
         8: new           #4                  // class DynamicDispatch$Woman
        11: dup
        12: invokespecial #5                  // Method DynamicDispatch$Woman."<init>":()V
        15: astore_2
        16: aload_1
        17: invokevirtual #6                  // Method DynamicDispatch$Human.sayHello:()V
        20: aload_2
        21: invokevirtual #6                  // Method DynamicDispatch$Human.sayHello:()V
        24: new           #4                  // class DynamicDispatch$Woman
        27: dup
        28: invokespecial #5                  // Method DynamicDispatch$Woman."<init>":()V
        31: astore_1
        32: aload_1
        33: invokevirtual #6                  // Method DynamicDispatch$Human.sayHello:()V
        36: return
      LineNumberTable:
        line 28: 0
        line 29: 8
        line 30: 16
        line 31: 20
        line 32: 24
        line 33: 32
        line 34: 36
}
SourceFile: "DynamicDispatch.java"
InnerClasses:
     static #9= #4 of #7; //Woman=class DynamicDispatch$Woman of class DynamicDispatch
     static #11= #2 of #7; //Man=class DynamicDispatch$Man of class DynamicDispatch
     static abstract #13= #12 of #7; //Human=class DynamicDispatch$Human of class DynamicDispatch

```

最上面的常量池就不用看了，0-15行是字节码的准备动作，目的就是生成下面这两行代码：

```java
Human man = new Man();
Human woman = new Woman();
```

16-21行就很关键了，16、20分别把刚才创建的两个对象的引用压到栈顶，这两个对象是要执行的sayHello方法的持有者，称为接受者(Receiver);17和21句是方法调用指令，这两条指令单从字节码角度来看，无论指令还是参数都完全一样，但是这两句指令最终执行的目标方法并不相同。原因就要从invokevirtul指令的多态查找过程说起，该指令在运行时解析过程如下所示：

1. 找到操作数栈顶的第一个元素所指向的对象的实际类型，记为C
2. 如果在类型C中找到与常量池中描述符和简单名称都相符的方法，则进行访问权限校验，如果通过，则返回这个方法的直接饮用，查找过程结束；如果不通过，则返回 java.lang.IIIegalAccessError异常。
3. 否则，按照继承关系从下往上依次对C的各个父类进行第二部的搜索和验证过程。
4. 如果始终没有找到合适的方法，则抛出java.lang.AbstractMethodError异常。


invokevirtual指令执行的第一步就是在运行期间确定接受者的实际数据类型，所以两次调用过程中的invokevirtual指令把常量池中的类方法符号引用解析到了不同的直接引用上，这个过程就是Java语言中方法重写的本质。把这种在运行期间根据实际类型确定方法执行版本的分派过程称为动态分派。

### 单分派与多分派

**方法的接受者**与**方法的参数**统称为方法的**宗量**，根据分派基于多少宗量将分派划分为单分派和多分派。看下面的的代码来理解一下这个概念：


```java
public class Dispatch {
    static class QQ{}

    static class _360{}

    public static class Father{
        void hardChoice(QQ arg){
            System.out.println("father choose qq");
        }

        void hardChoice(_360 arg){
            System.out.println("father choose 360");
        }
    }

    public static class Son extends Father{
        void hardChoice(QQ arg){
            System.out.println("son choose qq");
        }

        void hardChoice(_360 arg){
            System.out.println("son choose 360");
        }
    }


    public static void main(String[] args) {
        Father father = new Father();
        Father son = new Son();
        father.hardChoice(new _360());
        son.hardChoice(new QQ());
    }
}
```

上述代码运行结果为：

```java
father choose 360
son choose qq
```

编译阶段编译器的选择过程，也就是静态分派过程，在选择目标执行方法时，要根据两个宗量进行选择，所以Java语言的静态分派属于多分派类型。按照上面的Demo,程序选择目标方法有两点依据：一是静态类型是Father还是Son，二是方法参数是QQ还是360.这次选择的结果是产生了两条invokevirtual指令，其参数分别是常量池中指向Father.hardChoice(360)还是Father.hardChoose(QQ)方法的符号引用。

再看运行阶段虚拟机的选择，也就是动态分派的过程，因为只有一个宗量作为依据，所以Java语言的动态分派属于单分派类型。在执行“son.hardChoice(new QQ())”这句代码所对应的invokevirtuial指令时，由于编译器已经决定目标方法的签名必须为hardChoose(QQ),这个时候参数的静态类型、实际类型都对方法的选择造成不了任何影响。唯一可以影响虚拟机选择的因素只有此方法的接受者的实际类型是Father还是Son。

### 虚拟机动态分派的实现

由于动态分派是很频繁的动作，并且版本选择过程需要在类方法元数据中搜索合适的目标方法，动态分派的版本虚拟机基于性能的考虑不会大规模的进行方法的搜索，这种情况下最多见的优化就是为类在方法区中建立一个虚方法表和一个接口方法表，使用虚拟方发表索引来替代元数据查找以提高性能。

![](http://ww1.sinaimg.cn/mw690/006dXScfgy1fiz9nqnfs1j30l00e0dgm.jpg)

虚方法表中存储着各个方法的实际入口地址，如果某个方法在子类中没有被重写，那子类的虚方法表里面的地址入口和父类相同的方法地址入口是一致的，都是指向父类的实现入口。如果子类重写了这个方法，子类方法表中的地址将会替换为指向子类实现版本的入口地址，如上图中Son重写了Father的全部方法，因此son方法表没有指向Father类型的数据箭头，但是两者都没有重写来自Object的方法，所以他们的方法表中所有从Object继承来的方法都指向了Object的数据类型。方法表一般在类加载的连接阶段进行初始化。
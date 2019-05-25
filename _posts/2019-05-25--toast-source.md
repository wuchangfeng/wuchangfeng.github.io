---
layout: post
title: Toast 源码简要分析
date: 2019-05-25 10:00:09 +0800
categories: 
---

Toast的源码分析

本篇文章简要分析下Toast的源码。

```java
 Toast.makeText(this,"toast",Toast.LENGTH_LONG).show();
```

#### makeText()

该方法主要是 new 一个 Toast 对象，包含 Toast 的布局、mNextView、以及显示时间等。

#### show()

show 方法主要获取一个 INotificationManager 的 service 对象，并将 pkg、tn、mDuration 插入到系统 service 中的显示 toast 队列。至此我们需要知道 INotificationManager 具备的功能以及 TN 类是什么？另外，在这里也需要知道调用 show，不会立即将 Toast 显示出来，在这里先记住是**系统进程来控制 toast 的显示与消失的**。

#### Toast构造方法

因为在 show 方法中直接能够使用 TN 对象，猜测 Toast 构造方法中以及初始化 TN 类，验证确实如此，另外构造方法中也对 TN 的一些属性进行了初始化。

#### TN类

```java
private static class TN extends ITransientNotification.Stub
```

查看 TN 类发现其继承 ITransientNotification.Stub 知道 Toast 的核心逻辑离不开 Binder。在 TN 源码中首先初始化了 Toast 的 show 和 hide 的两个 runnable，至此知道 Toast 的 show 和 hide 逻辑依赖于线程(主线程 Handler 处理，设想在子线程中调用 Toast 会有什么问题)。另外 hide 和 show 的逻辑中查看源码，发现也是 windowManager 的 addView 和 removeView 那一套。原理本质上还是比较简单，具备 windowManager 的基础的话。

接着看到 TN 的构造函数，发现很多 windowManager 的信息，所以说 Toast 本质上也是 wm 的一种表现形式。

#### ITransientNotification.Stub

```java
package android.app;
oneway interface ITransientNotification {
    void show();
    void hide();
}
```

上述就是  ITransientNotification.Stub 的 aidl 文件，在 TN 类中也有上述两个方法的实现，在 binder 机制中，stub 应该是属于服务端提供的接口方法，远端进程可以通过 new TN 对象来调用 show 和 hide 的方法来自控制 Toast 的显示和消失。

#### [NotificationManagerService](http://androidxref.com/9.0.0_r3/xref/frameworks/base/services/core/java/com/android/server/notification/NotificationManagerService.java)

在前面说过，调用 Toast 的 show 方法后，是将当前 toast 对象插入到 service 的 Toast 队列中，那么插入到队列的过程主要做了哪些逻辑呢？1：判断是不是系统 Toast 、2：判断是不是当前要入队的 Toast 已经在队列中 3：将当前 Toast 设置为前台进程(Activity 销毁 Toast 也能显示) 4：Toast存储队列中进行出队操作

在当前类中，出队的过程中会同同时会调用 tn 的 callback 来控制 Toast 的显示，但是消失会通过 handler 的 sendMessage 来控制，以此达到控制 Toast 的显示时长的逻辑。

以上部分参考[Android Toast的源码分析](https://blog.csdn.net/wzy_1988/article/details/43341761)


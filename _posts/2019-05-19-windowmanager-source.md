---
layout: post
title: WindowManager 逻辑整理
date: 2019-05-19 08:03:33 +0800
categories: 
---

### WindowManager

Activity 类似一个管理者(生命周期与逻辑处理)，管理视图(Window、View)的添加与删除，以及它们之间的交互；
Window 的具体实现是 PhoneWindow，PhoneWindow 持有 DecorView 的实例。Window 通过 WindowManager 将 DecorView 加载到其中，并将 DecorView 交给 ViewRoot ，进行视图绘制以及其他交互。

WindowManager 提供的三个方法分别是：addView、updateViewLayout、removeView。顾名思义，其中 updateViewLayout 可以轻松实现日常中拖动悬浮窗的功能，监听 View 的 onTouchListener 事件更新 LayoutParams 即可。上述三个 ViewManager 方法都是针对 View 的，由此可见 Window 都不是真实存在的，以 View 的形式存在。

在实际使用中对 Window 的操作都必须通过 WindowManager(WM)来实现。WM 的接口实现类 WMI 以桥接模式将实现交给 WindowManagerGloal(WMG)，因此 WMG 是真正实现 addview、removeView 和 updateviewlayout 的类。

#### addView 的实现

* 1、检查参数合法性，如果是子 Window 还需要调整一些布局参数;
* 2、创建 ViewRootImpl 并将 View 添加到列表中，分为存储 Window 对应的 View、Window 对应的 ViewRootImpl，Window 对应的布局参数，正在被删除的 View 对象;
* 3、通过 ViewRootImpl(VRI) 来更新并完成 Window 的添加过程：

View 的绘制过程是通过 VRI 来完成的，setView 会通过 requestLayout 来实现异步刷新操作。之后通过 WindowSession 最终完成  Window 的添加过程，本质上 Window 的添加也是一个 IPC 过程，在 Session 会通过 WMS 来实现添加过程。


#### removeView 的实现：

* 1、同样交给 WMG，首先在 mRoot 列表中查找待删除 View 的索引;
* 2、通过 ViewRootImpl 的 die 方法来实现删除过程，发送删除新消息将删除的 View 添加到待删除的 View 列表中，其中分为同步删除和异步删除。真正删除的逻辑在 dispatchDetachedFromWindow 中;

#### updateViewLayout 的实现：

* 1、更新 View 自身的 params;
* 2、更新 ViewRootImpl 的 params;

整个过程也是交给 WMS 的 relayoutwindow 来实现的;


### Activity 的 window 创建过程

系统创建 Activity 所属的 Window 对象并为其设置回调接口，Window 对象的创建是通过 PolicyManager 的 MakeNewWindow 方法实现的。其在内部 new PhoneWindow。Window 创建完毕，但是 Activity 的视图是怎么附属到 Window 上面呢，由于 Activity 视图是 SetContentView 方法提供，理清内部逻辑，发现真正的实现还是在 PhoneWindow 中：

* 1、创建 DecorView，顶级 View，一般包含标题栏和内容栏，内容栏必须存在;
* 2、 View 添加到 DecoreView 的 mContentParent 中;
* 3、回调 Activity 的 onContentChanged 方法通知 Activity 视图已经发生改变;

注意：Window 被创建的时机是 Activity 的 attach 方法中，但是实际上真正能响应的时机还是要到 onResume 阶段。

#### Dialog 的 Window 创建过程：

* 1、创建 Window，该过程同 Activity 的 Window 创建过程；
* 2、初始化 DecorView 并将 Dialog 的视图添加到 DecorView 中；
* 3、将 DecorView 添加到 Window 中并显示；

注意：Dialog 的 Context 必须是 Activity，因为只有 Activity 包含 token，另外如果不是，可以将 Window type 类型设置为系统级别的。

### Toast 的 Window 创建过程：

见下一篇博客。





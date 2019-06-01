---
layout: post
title: Activity 启动流程简要分析
date: 2019-06-01 10:42:51 +0800
categories: 
---

#### system_server

Android 系统启动后已经启动了 Zygote，ServiceManager，system_server 等系统进程；ServiceManager 进程中完成了 Binder 初始化；system_server 进程中 ActivityManagerService，WindowManagerService，PackageManagerService 等系统服务在 ServiceManager 中已经注册；最后启动了 Launcher 桌面应用。整体开篇流程的话，点击 lanucher 图标会向 ServiceManager 查询 AMS 服务，ServiceManager 本质上是 Binder，所以又会向 system_server 查询 AMS 服务。

#### startActivityForResult

无论从 intent 方式启动 还是桌面点击图标启动 activity ，最后重载的都是 startActivityForResult 这个方法，该方法中最核心的目的是将 activity 的启动流程丢到 Instrumentation 中的 execStartActivity 方法。在此处注意下 checkStartActivityResult 的作用，见名知意其作用是检查 activity 的启动结果。

#### Instrumentation.execStartActivity

进入到 execStartActivity 中 activity 的启动流程又交给了 `ActivityManagerNative.getDefault`来执行 startActivity ，此时 根据继承以及 Binder 接口的关系，实际上 startActivity 的启动流程又交给了 ActivityManagerService。但是真正的操作对象是 ActivityManagerProxy ，这根是 AMS 的代理对象。在当前类中会检查是否存在符合要求的 activity 存在。

简要说明下，管理 activity 的接口是 IActivityManager，AMS 继承自 ActivityManagerNative(AMN)，而 AMN 继承自 Binder 并且实现了 IActivityManager 。另外在这其中 Instrumentation 使用来做应用程序和系统之间的桥梁。

#### AMS

AMS 的 startActivity 中要一串复杂的启动逻辑转移，开始从 ActivityStackSupervisor 到最后的 ActivityStack 中，这里面 AMS 的逻辑比较复杂，简单点说明最后startActivity 的逻辑转移到 ApplicationThread 中。

AMS 存在于 system_server 进程中，它会判断当前 activity 所在的进程到底存不存在，不存在就 fork 新进程。如果存在的话就通过 ApplicationThreadProxy 发送信号到 app 进程中的 ApplicationThread 请求启动 activity。

另外如果创建了新的 app 进程，最后也会回去向 system_server 进程中的 AMS 请求 attach 上去。这样 ApplicationThread 和 AMS 就可以不借助与其他进程进行通信了，通过 ApplicationThread 即可。后面的逻辑也跟上面一样。

#### ApplicationThread

ActivityThread（含ApplicationThread + ApplicationThreadNative + IApplicationThread），真正启动 Activity 的实现都在这里。ApplicationThread 继承自 ApplicationThreadNative( ATN),而 ATN 继承自 Binder 并且实现了 IApplicationThread 接口。应用进程需要调用 ActivityManagerService 提供的功能，反过来，ActivityManagerService 也需要调用应用进程的回调操作以达到控制和调配应用进程的目的。因此，AMS 也有一个 Binder 对象 ApplicationThreadProxy，它是 IApplicationThread 的代理对象。在 app 进程中，收到 BIND_APPLICATION_TRANSACTION 命令后调用 ActivityThread.bindApplication()，以此完成 app 进程的初始化。

#### **AcivityThread.ApplicationThread.scheduleLaunchActivity**

在该方法中，activity 的启动很简单，发送一个启动 activity 的方法给 handler( H ),在 handler 中的 handleMessage 中可以看见 'LANUCHER_ACTIVITY' 这个 case，发现 activity 的启动过程交给了 activitythread 的 handlerLanuchActivity 来实现。另外在该方法中也构造了 ActivityClientRecord 的实例，以及初始化了该实例的一些属性。ACR 的作用是记录 Activity 相关信息，比如：Window，configuration，ActivityInfo 等。 ActivityClientRecord 是客户端的，同样服务端也有对应的 ActivityRecord 是 ActivityManagerService 服务端的。

#### ActivityThread.handleLaunchActivity

performLaunchActivity 真正完成了 activity 的调起,同时 activity 会被实例化，并且 onCreate 会被调用，同时由于新 activity 的调起，原来的 activity 会处于 pause 状态，此处说原来的 activity 是因为启动 activity 要么从应用本身，要么从 lanucher，lanucher 也是一个 app。

##### ActivityThread. performLaunchActivity

在该方法中，主要完成以下几个事情：从 activityClientRecord 中获取待启动的 activity 的组件信息；通过 Instrumentation 的 newActivity 方法使用类加载器创建 activity 对象；通过 loadedApk 的 makeApplication 方法来尝试创建 Application 对象；创建 ContextImpl 对象并通过 Activity 的 attach 方法完成数据初始化；调用 activity 的 oncreate 方法。

另外在 activity 的 onresume 方法中会将 通过 system_server 中的 WMS 将 activity 与 window 相关联。

**本篇文章部分参考[一篇文章看明白 Android 从点击应用图标到界面显示的过程](https://blog.csdn.net/freekiteyu/article/details/79318031)**


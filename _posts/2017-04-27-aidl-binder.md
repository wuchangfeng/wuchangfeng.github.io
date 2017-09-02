---
layout: post
title: 深入理解 Android 之 Binder 工作机制
date: 2017-01-19 16:53:51 +0800
categories: android
---

我们从Android中的多进程通信开始来从Android层面深入到Binder机制。整个Binder机制太重要了，对于理解整个Android系统来说，Binder是其工作的核心，所以得要好好理解一下。

### Binder是啥

Binder是Android中的一个类，实现了IBinder接口，从IPC角度来说，Binder是Android中一种跨进程通信机制；从Android FrameWork角度来说，Binder是ServiceManager连接各种Manager(ActivityManager、WindowManager)和相应ManagerService的桥梁，从Android应用层来说Binder是客户端和服务端通信的媒介。其实说白了就是一种Android层面的通信机制，大家知道Androids是基于Linux层面的系统，Linux本身就存在多种IPC通信机制，为啥Android要自己提出一个通信机制呢，主要原因以下几点：

1：从性能的角度数据拷贝次数：Binder数据拷贝只需要一次，而管道、消息队列、Socket都需要2次，但共享内存方式一次内存拷贝都不需要；从性能角度看，Binder性能仅次于共享内存。
2：从稳定性的角度Binder是基于C/S架构的，简单解释下C/S架构，是指客户端(Client)和服务端(Server)组成的架构，Client端有什么需求，直接发送给Server端去完成，架构清晰明朗，Server端与Client端相对独立，稳定性较好；而共享内存实现方式复杂，没有客户与服务端之别， 需要充分考虑到访问临界资源的并发同步问题，否则可能会出现死锁等问题；从这稳定性角度看，Binder架构优越于共享内存。

后续的内容推荐你看[为什么Android采用Binder作为通信机制](https://www.zhihu.com/question/39440766/answer/89210950)

### 借助AIDL理解Binder

新建 AIDL 文件如下，并在其中定义方法 printMsg();

```java
interface IMyService {
    void printMsg(String message);
}
```

客户端的方法如下所示：

```java
public class MainActivity extends AppCompatActivity {
    private Button btn;

    private IMyService binder;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Intent intent = new Intent(this,MyRemoteService.class);
      // 让 activity 与 service 建立关联  
      bindService(intent,serviceConnection, Service.BIND_AUTO_CREATE);
        this.findViewById(R.id.btn).setOnClickListener(new View.OnClickListener(){
            @Override
            public void onClick(View v) {
                try {
                    binder.printMsg("this msg from client");
                } catch (RemoteException e) {
                    e.printStackTrace();
                }
            }
        });
    }
	// 主要用来获取 binder 对象。
    private ServiceConnection serviceConnection = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            binder = MyRemoteService.MyServiceBinder.asInterface(service);
        }

        @Override
        public void onServiceDisconnected(ComponentName name) {
            binder = null;
        }
    };
}
```

远程服务端的代码如下：

```java
public class MyRemoteService extends Service {
    private static final String TAG = "MyremoteService";

    private MyServiceBinder binder;

  	// 提供服务具体的方法实现
    public static class MyServiceBinder extends IMyService.Stub{
        @Override
        public void printMsg(String message) throws RemoteException {
            Log.d(TAG,"This message from remote" + message);
        }
    }

    @Override
    public void onCreate() {
        super.onCreate();
      // 获得 binder 对象
        binder = new MyServiceBinder();
    }

    public MyRemoteService() {
    }
	// onBind() 方法返回 binder 实例
    @Override
    public IBinder onBind(Intent intent) {
        // TODO: Return the communication channel to the service.
        //throw new UnsupportedOperationException("Not yet implemented");
        return binder;
    }
}
```

在上述代码中 Stub 其实就是 Binder 的子类的一个实例。

整个一个执行流程就是下面这样：

- 新建 AIDL 文件，并定义需要实现的方法/功能。
- 借助 RemoteService，在其内部实现 Binder 的子类 Stub，实现对应的方法。
- 借助 RemoteService 中的 onBind() 方法返回 binder 对象，即 Stub 的实例。
- 在客户端通过 ServiceConnection 方法获得 binder 实例。
- 通过 binder 实例调用 Stub 中实现的方法。

### 分析AIDL文件

下面的代码是 AIDL 机制在 rebuild project 之后自动生成的，可以到对应路径查看。Stub 类的代码如下所示。其实 Stub 的作用就是用来提供服务端的服务。具体点说stub作为一个Binder类可以判断：当客户端和服务端在同一个进程时，方法调用不会走跨进程的transact过程，当两者不在同一个进程时候，方法调用会走transact过程。这个具体的逻辑判断由stub的内部代理类proxy来完成。

```java
package com.allenwu.aidldemo;
// Declare any non-default types here with import statements

public interface IMyService extends android.os.IInterface
{
/** Local-side IPC implementation stub class. */
public static abstract class Stub extends android.os.Binder implements com.allenwu.aidldemo.IMyService
{
private static final java.lang.String DESCRIPTOR = "com.allenwu.aidldemo.IMyService";
/** Construct the stub at attach it to the interface. */
public Stub()
{
this.attachInterface(this, DESCRIPTOR);
}
/**
 * Cast an IBinder object into an com.allenwu.aidldemo.IMyService interface,
 * generating a proxy if needed.
 */
public static com.allenwu.aidldemo.IMyService asInterface(android.os.IBinder obj)
{
if ((obj==null)) {
return null;
}
android.os.IInterface iin = obj.queryLocalInterface(DESCRIPTOR);
if (((iin!=null)&&(iin instanceof com.allenwu.aidldemo.IMyService))) {
return ((com.allenwu.aidldemo.IMyService)iin);
}
// 需要跨进程，new 一个Proxy代理类出来
return new com.allenwu.aidldemo.IMyService.Stub.Proxy(obj);
}
@Override public android.os.IBinder asBinder()
{
return this;
}
```

我们在 serviceconnection 方法中获取 binder 实例就是通过 

```java
MyRemoteService.MyServiceBinder.asInterface(service);
```

来获取的。其实通过这一个步骤获取的 binder 对象其实就是 Binder 机制中的 binder 驱动。看下面一句话：

```java
return new com.allenwu.aidldemo.IMyService.Stub.Proxy(obj);
```

这样就很自然的将我们的 binder 对象/驱动传入到 Proxy 对象中去了。Proxy 类的代码如下：

```java
private static class Proxy implements com.allenwu.aidldemo.IMyService
{
private android.os.IBinder mRemote;
Proxy(android.os.IBinder remote)
{
mRemote = remote;
}
@Override public android.os.IBinder asBinder()
{
return mRemote;
}
public java.lang.String getInterfaceDescriptor()
{
return DESCRIPTOR;
}

@Override public void printMsg(java.lang.String message) throws android.os.RemoteException
{
  // 接受数据的 parcel 对象
android.os.Parcel _data = android.os.Parcel.obtain();
  // 传递数据的 parcel 对象
android.os.Parcel _reply = android.os.Parcel.obtain();
try {
_data.writeInterfaceToken(DESCRIPTOR);
_data.writeString(message);
  // 调用服务端的 transact 对象，其中这四个参数在下面分析过了
mRemote.transact(Stub.TRANSACTION_printMsg, _data, _reply, 0);
_reply.readException();
}
finally {
_reply.recycle();
_data.recycle();
}
}
}
static final int TRANSACTION_printMsg = (android.os.IBinder.FIRST_CALL_TRANSACTION + 0);
}
/**
     * Demonstrates some basic types that you can use as parameters
     * and return values in AIDL.
     *///void basicTypes(int anInt, long aLong, boolean aBoolean, float aFloat,
//        double aDouble, String aString);

public void printMsg(java.lang.String message) throws android.os.RemoteException;
}
```

onTransact 运行在服务端的Binder线程池中，其代码如下所示

```java
@Override public boolean onTransact(int code, android.os.Parcel data, android.os.Parcel reply, int flags) throws android.os.RemoteException
{
switch (code)
{
case INTERFACE_TRANSACTION:
{
reply.writeString(DESCRIPTOR);
return true;
}
case TRANSACTION_printMsg:
{
data.enforceInterface(DESCRIPTOR);
java.lang.String _arg0;
  // 读入传进来的数值
_arg0 = data.readString();
  // 回调我们实现的方法
this.printMsg(_arg0);
reply.writeNoException();
return true;
}
}
return super.onTransact(code, data, reply, flags);
}
```

服务端的 Binder 实例会根据客户端依靠 Binder 驱动发来的消息，执行 onTransact 方法，根据其参数执行服务端的代码，解释一下四个参数具体的意思：

1. code 用于区分执行哪个方法，客户端会传递此参数，告诉服务端执行具体方法
2. data 即客户端传过来的参数
3. replay 服务端返回的数值
4. flag 标明是否有返回值

其实 Service 就是提供一个服务端与客户端通信的桥梁，称之为驱动 binder。

### 文字描述

1. 首先，Server进程要向ServiceManager注册；告诉自己是谁，自己有什么能力；在这个场景就是Server告诉SM，它叫zhangsan，它有一个object对象，可以执行add 操作；于是SM建立了一张表：zhangsan这个名字对应进程Server;

2. 然后Client向SM查询：我需要联系一个名字叫做zhangsan的进程里面的object对象；这时候关键来了：进程之间通信的数据都会经过运行在内核空间里面的驱动，驱动在数据流过的时候做了一点手脚，**它并不会给Client进程返回一个真正的object对象**，而是返回一个看起来跟object一模一样的代理对象objectProxy，这个objectProxy也有一个add方法，**但是这个add方法没有Server进程里面object对象的add方法那个能力**；objectProxy的add只是一个傀儡，它唯一做的事情就是把参数包装然后交给驱动。

3. 但是Client进程并不知道驱动返回给它的对象动过手脚，毕竟伪装的太像了，如假包换。Client开开心心地拿着objectProxy对象然后调用add方法；我们说过，这个add什么也不做，直接把参数做一些包装然后直接转发给Binder驱动。

4. 驱动收到这个消息，发现是这个objectProxy；一查表就明白了：我之前用objectProxy替换了object发送给Client了，它真正应该要访问的是object对象的add方法；于是Binder驱动通知Server进程，调用你的object对象的add方法，然后把结果发给我，Sever进程收到这个消息，照做之后将结果返回驱动，驱动然后把结果返回给Client进程；于是整个过程就完成了。

5. 由于驱动返回的objectProxy与Server进程里面原始的object是如此相似，给人感觉好像是直接把Server进程里面的对象object传递到了Client进程；因此，我们可以说Binder对象是可以进行跨进程传递的对象

6. 但事实上我们知道，Binder跨进程传输并不是真的把一个对象传输到了另外一个进程；传输过程好像是Binder跨进程穿越的时候，它在一个进程留下了一个真身，在另外一个进程幻化出一个影子（这个影子可以很多个）；Client进程的操作其实是对于影子的操作，影子利用Binder驱动最终让真身完成操作。

理解这一点非常重要；务必仔细体会。另外，Android系统实现这种机制使用的是代理模式, 对于Binder的访问，如果是在同一个进程（不需要跨进程），那么直接返回原始的Binder实体；如果在不同进程，那么就给他一个代理对象（影子）；我们在系统源码以及AIDL的生成代码里面可以看到很多这种实现。

![](http://ww1.sinaimg.cn/mw690/006dXScfly1fizes8isdej30ix0dhdgo.jpg)


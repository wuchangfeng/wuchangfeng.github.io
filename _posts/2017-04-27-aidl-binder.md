---
layout: post
title: 从 AIDL 机制来理解 Binder
date: 2017-01-19 16:53:51 +0800
categories: android
---

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

下面的代码是 AIDL 机制在 rebuild project 之后自动生成的，可以到对应路径查看。Stub 类的代码如下所示。其实 Stub 的作用就是用来提供服务端的服务。

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

onTransact 代码如下所示

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
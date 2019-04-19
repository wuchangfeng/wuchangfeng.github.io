---
layout: post
title: Android O 版本适配
date: 2019-04-19 20:28:34 +0800
categories: 
---

最近项目要适配华为市场的 Android Q，为了避免升级跨度过大，我们先从原来的 targetSdkVersion 从 23 升级到 26 了，即适配过程从 Android N 到 Android O ，下面记录下适配过程中遇到一些问题。

### 应用快捷方式创建的改变

Android 8.0 对应用快捷方式做出了以下变更：` com.android.launcher.action.INSTALL_SHORTCUT`
 广播不再会对您的应用有任何影响，因为它现在是私有的隐式广播。相反，您应使用 `ShortcutManager`类中的 `requestPinShortcut`函数创建应用快捷方式。以下是 Android O 以及之后 Android 系统创建快捷方式的逻辑：

```java
ShortcutManager shortcutManager = (ShortcutManager) context.getSystemService(Context.SHORTCUT_SERVICE);
            if (null == shortcutManager) {
                return;
            }
            ShortcutInfo shortcutInfo = new ShortcutInfo.Builder(context, shortcutName)
                    .setShortLabel(shortcutName)
                    .setIcon(Icon.createWithResource(context, resourceId))
                    .setIntent(getShortcutIntent(shortcutClass))
                    .setLongLabel(shortcutName)
                    .build();
            shortcutManager.requestPinShortcut(shortcutInfo, PendingIntent.getActivity(context,
                    0, getShortcutIntent(shortcutClass), PendingIntent.FLAG_UPDATE_CURRENT).getIntentSender());
```

### FileProvider机制的出现(Android N)

Android N 在 App 间对 `file://` 的分享做了严格的校验，所有包含 `file://` 的 URI 的 Intent 离开开发 App (调用相机拍照`,`剪裁图片`,`调用系统安装器安装 Apk)，传递一个 File ，都会抛出 FileUriExposedException 的错误，并且引发 Crash。实际上也提供了解决方案，那就是 FileProvider，通过 `comtent://` 的模式替换掉 `file://` ，同时，需要开发者主动升级 targetSdkVersion 到 24 才会执行此策略，也留给了开发者升级的时间。

不过，如果实在来不及或者只需要渠道适配的话，可以采用以下取巧模式，在基类 onCreate() 方法中添加即可，具体可以参考[stackoverflow 这个回复](https://stackoverflow.com/questions/44821017/fileuriexposedexception-using-android-7)：

```java
StrictMode.VmPolicy.Builder builder = new StrictMode.VmPolicy.Builder();
StrictMode.setVmPolicy(builder.build());
```

### 隐私广播

由于 Android 8.0 引入了新的广播接收器限制，因此您应该移除所有为隐式广播 Intent 注册的广播接收器。将它们留在原位并不会在构建时或运行时令应用失效，但当应用运行在 Android 8.0 上时它们不起任何作用。显式广播 Intent（只有您的应用可以响应的 Intent）在 Android 8.0 上仍以相同方式工作。类似自己应用里定义的广播也无法被静态注册的 BroadcastReceiver 收到了。比较简单的一种就是修改静态广播为显示广播：

```java
intent.action = "com.example.test.CLOSE”;
intent.setPackage(getPackageName()); // 指定包名
context.sendBroadcast(intent);
```

### 网络变化监听(Android N)

Android N 移除了三项隐式广播，以帮助优化内存使用和电量消耗。此项变更很有必要，因为隐式广播会在后台频繁启动已注册侦听这些广播的应用。删除这些广播可以显著提升设备性能和用户体验。

移动设备会经历频繁的连接变更，例如在 WLAN 和移动数据之间切换时。目前，可以通过在应用清单中注册一个接收器来侦听隐式 `CONNECTIVITY_ACTION` 广播，让应用能够监控这些变更。由于很多应用会注册接收此广播，因此单次网络切换即会导致所有应用被唤醒并同时处理此广播。

同理，在之前版本的 Android 中，应用可以注册接收来自其他应用（例如相机）的隐式 `ACTION_NEW_PICTURE` 和 `ACTION_NEW_VIDEO` 广播。当用户使用相机应用拍摄照片时，这些应用即会被唤醒以处理广播。

以下逻辑用于适配 Android N 之后及时监听系统网络变化的逻辑：

```java
// 基类
private MainActivity extend Activity{
  private void onCreate(){
    networkCallback = new NetworkCallbackImpl();
        NetworkRequest.Builder builder = new NetworkRequest.Builder();
        NetworkRequest request = builder.build();
        connectivityManager = (ConnectivityManager) getSystemService(Context.CONNECTIVITY_SERVICE);
        connectivityManager.registerNetworkCallback(request, networkCallback);
  }
  
  private void onDestory(){
    connectivityManager.unregisterNetworkCallback(networkCallback);
  }
}

// 网络变化监听类
private class NetworkCallbackImpl extends ConnectivityManager.NetworkCallback {
  @Override
    public void onAvailable(Network network) {
        super.onAvailable(network);
      // 发送变化事件
        postNetStateEvent();
    }

    @Override
    public void onLosing(Network network, int maxMsToLive) {
        super.onLosing(network, maxMsToLive);
    }

    @Override
    public void onLost(Network network) {
        super.onLost(network);
      	// 发送变化事件
        postNetStateEvent();
    }

    @Override
    public void onCapabilitiesChanged(Network network, NetworkCapabilities networkCapabilities) {
        super.onCapabilitiesChanged(network, networkCapabilities);
    }

    @Override
    public void onLinkPropertiesChanged(Network network, LinkProperties linkProperties) {
        super.onLinkPropertiesChanged(network, linkProperties);
    }
}

```

### 读写权限问题

在 Android 8.0 之前，如果应用在运行时请求权限并且被授予该权限，系统会错误地将属于同一权限组并且在清单中注册的其他权限也一起授予应用。对于针对 Android 8.0 的应用，此行为已被纠正。系统只会授予应用明确请求的权限。然而，一旦用户为应用授予某个权限，则所有后续对该权限组中权限的请求都将被自动批准。

例如，假设某个应用在其清单中列出 `READ_EXTERNAL_STORAGE` 和 `WRITE_EXTERNAL_STORAGE`。应用请求 `READ_EXTERNAL_STORAGE`，并且用户授予了该权限。如果该应用针对的是 API 级别 24 或更低级别，系统还会同时授予 `WRITE_EXTERNAL_STORAGE`，因为该权限也属于同一 `STORAGE` 权限组并且也在清单中注册过。**如果该应用针对的是 Android 8.0，则系统此时仅会授予 `READ_EXTERNAL_STORAGE`；不过，如果该应用后来又请求 `WRITE_EXTERNAL_STORAGE`，则系统会立即授予该权限，而不会提示用户。**

### 悬浮窗权限问题

如果项目中有使用到悬浮窗并且进行相应的悬浮窗权限适配，则极易出现关键字为 `  android.view.WindowManager$BadTokenException` Crash 日志。使用 `SYSTEM_ALERT_WINDOW` 权限的应用无法再使用以下窗口类型来在其他应用和系统窗口上方显示提醒窗口：`TYPE_PHONE`,`TYPE_PRIORITY_PHONE`,`TYPE_SYSTEM_ALERT`,`TYPE_SYSTEM_OVERLAY`，`TYPE_SYSTEM_ERROR`。相反，应用必须使用名为 `TYPE_APPLICATION_OVERLAY` 的新窗口类型。该情形适配较简单，做以下区别即可：

```java
if (Build.VERSION.SDK_INT >= 26) {
    mWindowParams.type= WindowManager.LayoutParams.TYPE_APPLICATION_OVERLAY;
} else{
    mWindowParams.type= WindowManager.LayoutParams.TYPE_SYSTEM_ALERT;
}
```

### 通知栏

Android 8.0 引入了通知渠道，其允许您为要显示的每种通知类型创建用户可自定义的渠道。用户界面将通知渠道称之为通知类别。针对 Android 8.0 的应用，创建通知前需要创建渠道，创建通知时需要传入 channelId，否则通知将不会显示。示例代码如下：

```java
// 创建通知渠道
private void initNotificationChannel() {
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
        CharSequence name = mContext.getString(R.string.app_name);
        NotificationChannel channel = new NotificationChannel(mChannelId, name, NotificationManager.IMPORTANCE_DEFAULT);
        mNotificationManager.createNotificationChannel(channel);
    }
}
// 创建通知传入channelId
NotificationCompat.Builder builder = new NotificationCompat.Builder(context, NotificationBarManager.getInstance().getChannelId());
```

### 允许安装未知来源的应用

为了能够在应用内进行App的升级以及下载其他Apk进行安装，需要显示在 AndroidMainFest.mxl 文件中注册以下权限：

<uses-permission android:name="android.permission.REQUEST_INSTALL_PACKAGES" />

### 后台服务限制

如果针对 Android 8.0 的应用尝试在不允许其创建后台服务的情况下使用 startService() 函数，则该函数将引发一个 IllegalStateException。因而所有的 startService() 都有必要进行安全启动，即 Try-Catch 大法。
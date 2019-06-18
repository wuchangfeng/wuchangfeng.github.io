---
layout: post
title: 解决 Android Q 平台下应用内安装 Apk 的问题
date: 2019-06-12 22:02:00 +0800
categories: 
---

前期为了解决 Android N 的文件存储 FileUriExposedException 问题，在项目的 App 中设置了如下代码：

```java
  if (Build.VERSION.SDK_INT >= 24) {
            StrictMode.VmPolicy.Builder builder = new StrictMode.VmPolicy.Builder();
            StrictMode.setVmPolicy(builder.build());
  }
```

用了这种偷懒的方式，在 Android N、O、P 平台上进行 App 内的分享和应用内安装都没问题。但是适配 Android Q 时，应用不停的 Crash，主要路径是应用内下载 Apk ，安装过程出现问题，Error 日志如下：

```java
android.content.ActivityNotFoundException: No Activity found to handle Intent { act=android.intent.action.VIEW dat=file:///storage/emulated/0/com.xxx.xxx/cache/xxx.apk
```

首先根据日志的关键字去 Google，搜索了半天，虽然最后的结果是一样的 Crash，但是出现的原因各有不同，总结下来主要有以下几点：

* 1: Activity 没有注册或者根本没有该 Activity 
* 2：想要调整的应用本地不存在
* 3：uri 的文件格式不对
* 4：没有访问 uri 对应的文件的权限

如果是一般的别的 sdk 问题，无法更改其错误代码的话，可以在 try-catch 的基础上增加兜底方案。 

排除前面几点问题后，尤其是 uri 格式上的问题。开始认真考虑是不是访问文件的权限问题。联想到之前偷懒的方式会不会 Google 在最新的 Android 版本中禁止了，开始用官方推荐的 FileProvider 模式来解决 FileUriExposedException 问题。

当然，用官方的方式解决问题，也要对版本进行区分。出去一些 file_path 和 AndroidManiFest 文件的配置，在 Java 中进行如下适配：

```java
  Intent intent = new Intent(Intent.ACTION_VIEW);
  intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
        if (Build.VERSION.SDK_INT >= 24) {
            Uri apkUri = FileProvider.getUriForFile(context, "com.xxx.xxx.fileprovider", new File(filePath));
            intent.addFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION);
            intent.setDataAndType(apkUri, "application/vnd.android.package-archive");
        } else {
            intent.setDataAndType(Uri.fromFile(new File(filePath)), "application/vnd.android.package-archive");
        }
        try {
            context.startActivity(intent);
        } catch (Exception e) {
            e.printStackTrace();
        }
```

代码写完，编译，发现问题完美解决。在此验证，应该是 Android Q 的沙箱机制产生的影响。


---
layout: post
title: Android 工程中集成 Flutter Module
date: 2018-10-20 22:25:08 +0800
categories: 
---

为了将 Flutter 引入到 Android 工程中，官方提供了以 Module 形式载入。但是实践一下，发现坑真的很多。首先如果
需要实现这个需求，可以参照 [Add Flutter to Android](https://github.com/flutter/flutter/wiki/Add-Flutter-to-existing-apps#experiment-turn-the-flutter-project-into-a-module) 这篇官方wiki来实现。下面我讲述一下遇到的坑吧，算是一个记录。

### Module 的位置

Android 开发者很容易误以为集成 Flutter Module 就像以前 Android 工程集成 Module 一样，所以按照这种思路，执行依赖 Sync 的时候就会出现 `include_flutter.groovy` 相关文件丢失问题。

要命的是这种问题是非常低级的，低级到别人都没遇到过。解决这种问题两种思路，第一种情况，本质就是你的配置错了，官方 wiki 让你将 flutter_module 放在 Android 工程的`同级目录`下，而不是 Android 工程目录下。想一下这种问题为什么会出现，主要就是一些不好的文章翻译不正确导致的问题。

第二种就是尝试一下在出错的地方理解下原理，路径配置的时候，加上 Android 工程的路径，试验下发现也是可以 Sync 过的。

```
setBinding(new Binding([gradle: this]))                                 
evaluate(new File(                                                      
        settingsDir.parentFile,                                               
        '/工程路径/my_flutter/.android/include_flutter.groovy'                          
))
```

### support 依赖包冲突

Flutter 中 Android 工程可能会与项目中的 support 之间存在冲突，加入如下代码即可：

```
dependencies {
    ...
    implementation(project(':flutter'),{
        exclude group: 'com.android.support'
    })
    ...
```

### minSdkVersion

这个比较简单，升级一下你的 minSdkVersion 的版本要是16

### getLifeCycle 找不到

添加谷歌Maven仓库，默认情况下，Android Studio 项目是不会配置的。如果要加入此仓库，打开项目的 `build.gradle`，将下面高亮的部分加入到配置文件中。

```
allprojects {
    repositories {
        jcenter()
        maven { url 'https://maven.google.com' }
    }
}
```

同时 Android 工程的 buildToolsVersion 太低，将25改成27。

### 验证

最后，在 Android 宿主工程的 MainActivity 的 onCreate 方法中，加入以下代码，验证是否能够成功调起 Flutter 对应的页面：

``` Dart
        View flutterView = Flutter.createView(
                MainActivity.this,
                getLifecycle(),
                "route1"
        );
        FrameLayout.LayoutParams layout = new FrameLayout.LayoutParams(600, 800);
        layout.leftMargin = 100;
        layout.topMargin = 200;
        addContentView(flutterView, layout);

```

以上就是按照官方 Wiki 配置会出现的一些问题以及最后验证是否配置成功的代码。

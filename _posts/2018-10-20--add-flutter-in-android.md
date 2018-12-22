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

### Gradle 版本升级

gradle 版本要升级到 3.xx 吧，如果你的项目 gradle 版本是 2.xx之类。由此首要带来的的就是 compile 要改成implemention，本质上带来的好处是分包编译，加快编译速度。但是如果你的项目有多个 module 的话，并且 subModule要为 mainModule 提供 api 等，注意需要将 compile 改成 api. 另外，也要注意包含 apt 的依赖，如 eventbus 也要改一下依赖方式。

### so 库的兼容性

这个坑，如果没有一定经验真的很难发现。flutter 最后打包出的 apk 解压出来发现包含有 libflutter.so 包。然而为了兼容性的提高，flutter 会在四个不同类型的目录下放置 so 包，这就很很容易导致问题，因为大部分项目为了减少包大小以及追求合理性的兼容，只会有一个类似 armeabi 或者 armeabi-v7a 等。如此，解决方式就是修改 flutter.gradle 脚本中的相应 so 包注入代码，使其只注入特定目录下。这一点可以参考美团的 flutter 文章，其中有对应脚本实现。

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

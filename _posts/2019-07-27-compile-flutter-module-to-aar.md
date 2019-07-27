---
layout: post
title: Android 项目直接依赖 Flutter Module AAR
date: 2019-07-27 09:59:29 +0800
categories: 
---

为了降低 Flutter 对于原生项目的侵入性，方便项目组里 Native 开发人员编译开发，考虑将 Flutter Module 开发的跨平台功能模块单独抽离打包为 AAR，Google 一番结合实际操作，记录要点如下：

#### 1：创建 Flutter Module 开发跨平台端的功能

这里就按照 Flutter Project 来开发就好了，不用跟 native 工程一起编译，比较方便的体验热重载功能。

#### 2：打包 Flutter Module 成为 AAR 文件

android 目录下 app 层级下：

```groovy
def isLib = true
if(isLib) {
    apply plugin: 'com.android.library'
} else  {
    apply plugin: 'com.android.application'
}

apply from: "$flutterRoot/packages/flutter_tools/gradle/flutter.gradle"

if(isLib) {
    apply plugin: 'com.kezong.fat-aar'
}

android {
    compileSdkVersion 28
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }

    if (isLib){
        resourcePrefix "flutter_module"
    }

    lintOptions {
        disable 'InvalidPackage'
    }

    defaultConfig {
        if(!isLib) {
            applicationId "com.example.flutter_aar_test"
        }
        minSdkVersion 16
        targetSdkVersion 28
        versionCode flutterVersionCode.toInteger()
        versionName flutterVersionName
        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"

        if(isLib) {
            ndk {
                // 设置支持的SO库架构
                abiFilters  'armeabi-v7a'
            }
        }
    }
}

flutter {
    source '../..'
}

dependencies {
	// ...
    if(isLib) {
        def flutterProjectRoot = rootProject.projectDir.parentFile.toPath()
        def plugins = new Properties()
        def pluginsFile = new File(flutterProjectRoot.toFile(), '.flutter-plugins')

        if (pluginsFile.exists()) {
            pluginsFile.withReader('UTF-8') { reader -> plugins.load(reader) }
        }

        plugins.each { name, _ ->
            println name
            embed project(path: ":$name", configuration: 'default')
        }
    }
}
```

Project 层级下的 build.gradle 如下

```java
 dependencies {
       classpath 'com.android.tools.build:gradle:3.2.1'
       classpath 'com.kezong:fat-aar:1.1.10'
  } 
```

#### 3: 解压 AAR 文件对齐 so 类型

AAR 文件中解压出的 so 类型，要跟 native 项目中的 so 类型一致，否则会出现错误，建议统一为 armeabi-v7a 类型。

#### 4：native 端 调用 

Android 端桥接页面，直接导航至 Flutter 的默认路由界面：

```java
public class FlutterBridgeActivity extends FlutterActivity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        GeneratedPluginRegistrant.registerWith(this);
    }
}
```

跳转至导航页面之前的初始化：

```java
Flutter.startInitialization(this.getApplicationContext());
Intent intent = new Intent(this, FlutterBridgeActivity.class);
startActivity(intent);
```

#### 5：容易出现的问题

主要是打包 AAR 文件时，对于 Plugin 的引入，之前以为 Plugin 所有的内容以及依赖都包含在 Plugin 产物中，但是实际上不是，Plugin 的一些 compile 依赖并不会包含在其中，所以在 Native 引用 Flutter AAR 时要注意，如果没有对应的类和文件，要在 Native Project 中自己加上依赖。
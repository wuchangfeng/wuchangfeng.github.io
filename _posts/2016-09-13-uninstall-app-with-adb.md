---
title: 利用 Python 脚本结合 ADB 命令来卸载 App 
toc: true
date: 2016-06-22 22:09:37
tags: python,adb,as
categories: About Java
description:
feature:
---

一个 Python 脚本,用来批量卸载模拟器或者实体机上面的 App 以及清除 LogCat 缓存。

<!--more-->

开发 Android 的朋友,模拟器或者手机里面常常有大量调试的 Demo，对于手机来说还好，可是对于模拟器，有可能就会造成调试速度以及启动速度的下降。而且模拟器中 App 一个一个删除也是很麻烦。利用 ADB 命令，我们可以做很多事，其中就包括批量操作模拟器或者手机上的 App。当然包括删除操作啦。利用 Python 脚本和 ADB shell 命令以及 AS 自带的 CMD 窗口,我们就可以将这一切浓缩成一个命令行啦。

### 核心代码

``` python
# 删除所有你指定包名的 APP
def delAllapp( ):
    print 'start delete all your app in your Phone or Simulator '
    os.popen('adb wait-for-device');
    corename = raw_input("input your app package corename:")
    oriPackages = os.popen('adb shell pm list packages {name}'.format(name=corename));
    # list all PackageName
    for oriPackage in oriPackages:
        deletePackage = oriPackage.split(':')[1]
        os.popen('adb uninstall ' + deletePackage );
        print deletePackage + "is deleted"
        
# 删除所有你指定包名的特定 APP
def listAllpackage( ):
    i = 0
    os.popen('adb wait-for-device');
    corename = raw_input("input your app package corename:")
    oriPackages = os.popen('adb shell pm list packages {name}'.format(name=corename));
    
    for oriPackage in oriPackages:
        deletePackage = oriPackage.split(':')[1]
        print str(i) + ":" + deletePackage
        deleteList.append(deletePackage)
        i += 1

# 删除指定 App
def deleteApp(number):
    os.popen('adb uninstall ' + deleteList[number] );
    print 'delete '+ deleteList[number] + "success"
 
# 清除 LogCat 缓存   
def clearLogcat( ):
    print 'start clear logcat buffer in your Phone or Simulator'
    os.popen('adb wait-for-device');
    os.popen('adb logcat -c');
    print 'logcat is cleared success'       
    
```

### 效果如下

![img](http://7xrl8j.com1.z0.glb.clouddn.com/Use.gif)

### 使用方式

- 确保你的 AS 能够使用 ADB 命令
- 配置 Python 2.7 环境(3+ 应该也没有问题)
- 在 AS 提供的 CMD 中找到当前脚本路径 输入: python unistall.py
- 根据命令提示输入你想要删除 App 的包的核心关键字，如:com.example.RxCacheDemo ,输入 example 即可(每个人 AS 的这个配置应该都是一样的)
- 以上步骤完成之后会有提示 删除成功与否。

**当然,脚本还可以指定具体应用进行删除**,你只需要去掉注释以及注释调用现有函数的代码即可。

### 相关扩展

- [ADB shell 命令](http://imsardine.simplbug.com/note/android/adb/commands/pm.html)

- [ADB 常用命令](https://segmentfault.com/a/1190000000426049)

- [Python ADB 命令](http://www.cnblogs.com/HQMIS/archive/2013/02/03/2890892.html)

  ​
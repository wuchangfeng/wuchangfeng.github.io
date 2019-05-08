---
layout: post
title: 用于批量卸载 App 的 Pypi 库
date: 2018-01-20 16:55:13 +0800
categories: 
---

之前我写过一个 Py 脚本，用来删除模拟器/手机上的 App，因为测试机 App 太多了，一个一个删除，显得非常傻瓜。一直想着把之前写过的一个 Py 脚本封装一下，打包成 Pypi 库，更方便的使用。尤其是最近模拟器和手机里面的 App 数量暴增。找了一篇如何打包 Pypi 的[教程](https://zhuanlan.zhihu.com/p/26159930)基本上是流程式操作，跟着教程一步一步来就行了。一切完成之后，你的本地文件目录是这样的：

![](http://ww1.sinaimg.cn/large/b10d1ea5ly1fez6xha891j216s0koteh.jpg)

如果没有出现什么错误，你就可以在 Python 的库中心看见你的库了：

![](http://ww1.sinaimg.cn/large/b10d1ea5ly1fez6zacgm0j21gu0r2grz.jpg)

最后，这个库的使用非常爽，如果你第一次使用，按照如下步骤即可。在 IDE 中自带的命令行：

```python
pip install uninstall_app
```

源代码提供了三个功能，第一个为 `cl`,输入之后，会提示你输入要删除 app 的包名前缀：

```python
$  cl
```

第二个命令 ca，可以删除你的模拟器或者手机里面所有的第三方 app。我比较看重这个功能，因为我调试的情况下大部分是用模拟器来进行的。

```python
$  ca
```

还有一个清除 logcat 缓存的命令。这个也是很重要的，可以清空 logcat 缓存。

```python
$  cc
```

大概就是这么多，很简单的一个小东西，完整一下，还是挺有必要的。不过，千万不要随意在你的测试手机上使用，因为常用的命令是删除第三方 App，我专门写着为模拟器用的。


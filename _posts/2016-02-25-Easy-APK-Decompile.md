---
title: 超级简单的 APK 反编译方法
---

虽然反编译 APK 在某些情况下显得不道义，但是掌握反编译的技巧还是有必要的。

一般来说 APK 反编译工具有 [Apktool](https://github.com/iBotPeaches/Apktool "iBotPeaches/Apktool") 、 [dex2jar](https://github.com/pxb1988/dex2jar "pxb1988/dex2jar") 、[jd-gui](https://github.com/java-decompiler/jd-gui "java-decompiler/jd-gui") 等。

我没有用过 Apktool ，对它不做评价。 dex2jar 一般和 jd-gui 搭配使用， dex2jar 将 APK 中编译生成的 dex 文件转换为 jar ，接着使用 jd-gui 打开 jar ，就可以看见反编译后的代码了。

经过代码混淆的 APK ，反编译生成的代码可读性自然不高。但是 dex2jar/jd-gui 在对（几乎）没有混淆的代码进行反编译时，效果也不是很理想，部分代码可能无法显示；而且整体的操作过程也略显繁琐。所以在这里我推荐大家一款**超级好用**的反编译工具， [jadx](https://github.com/skylot/jadx "skylot/jadx") 。

jadx 可以直接打开 APK 包，自行生成对应的反编译代码，相当的傻瓜式；而且生成的代码非常漂亮，在我的使用过程中（几乎）没有出现过反编译不出来的代码部分。对于使用了 [MultiDex](http://developer.android.com/intl/zh-cn/tools/building/multidex.html "Building Apps with Over 65K Methods") 的 APK 来说，直接使用 jadx 打开，只会显示第一个 dex 文件的内容。不过没关系，你可以将该 APK 解压，使用 jadx 分别打开 dex 文件，就没有问题了。

当然反编译工具自带的搜索功能都比较弱鸡，肯定不能和 IDE 比。为了方便检索我们需要的代码，建议将反编译生成的代码保存在一个文件夹下，在该文件夹中使用 `grep -rn STRING` 命令进行检索，相当地方便。

以反编译 Yalantis 的 [uCrop](https://github.com/Yalantis/uCrop "Yalantis/uCrop") demo 为例。为方便演示，在这里使用 jadx 的命令行版本（你也可以使用 jadx 的 GUI 版本）。

首先将 uCrop.apk 反编译到 out 目录， jadx 在运行过程中会有相关的输出信息，不过基本可以忽略：

![jadx.png](/assets/img/2016-02-25-jadx.png "jadx")

接着切换到 out 目录，我们想查看 uCrop 中有哪些地方使用到了 `Context.startActivity()` 方法，只需要输入 `grep -rn "startActivity"` 即可：

![grep.png](/assets/img/2016-02-25-grep.png "grep")

剩下的工作就是到对应包名下查看相关代码就可以啦。

怎么样，超级简单吧！
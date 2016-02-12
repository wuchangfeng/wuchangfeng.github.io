---
title: Chrome Overview Screen 分组效果的实现
---

从 Lollipop 开始， Android 提供了全新的多任务切换界面，叫做 [Overview Screen](http://developer.android.com/guide/components/recents.html "Overview Screen") ，想必大家都不陌生。

通常来说加入到 Overview Screen 的 Activities 都是单个独立的，但是 Chrome 不一样。如果你在 Chrome 中开启了「一并显示标签页和应用」，在标签页中长按链接，在上下文菜单选择「在新标签页中打开」，进入 Overview Screen 的时候你会发现，这两个的标签页是紧贴在一起的，呈现出一种「分组」的效果，就像下图所示一样：

![Overview.png](/assets/img/2016-02-12-Overview.png "Overview.png")

这个效果非常有用，对于同样需要将自己的多个 Activities 加入到 Overview Screen 中的 App 来说，辨识度更高。那么 Chrome 是怎么做到的呢？其实非常简单，只需要按照如下代码启动你的 Activity 即可：

{% highlight java %}
startActivity(newDocumentIntent,
        ActivityOptions.makeTaskLaunchBehind().toBundle());{% endhighlight %}

大家可以结合 [googlesamples/android-DocumentCentricApps](https://github.com/googlesamples/android-DocumentCentricApps "googlesamples/android-DocumentCentricApps") 的示例代码自己实验。

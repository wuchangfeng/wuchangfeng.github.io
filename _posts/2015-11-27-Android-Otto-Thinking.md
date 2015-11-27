---
title: 使用 Otto 的一些思考
---

[Otto](http://square.github.io/otto/ "Otto") 是 Square 公司开发的[事件总线](https://zh.wikipedia.org/wiki/%E5%8F%91%E5%B8%83/%E8%AE%A2%E9%98%85 "发布/订阅")库，想必大家或多或少都有使用过。

根据我自己的使用情况来看，至少有 2 个地方可以使用：

 1. Activity 与 Fragment 相互通信，甚至 View 与 View 之间相互通信。

 2. 作为网络操作的回调与界面之间的桥梁，解决 Android 生命周期导致的 OOM/NPE 等问题。

总体来说 Otto 还是很好用的。但最近在使用 Otto 的过程中，我遇到了一件麻烦事。

简单来说是这样的，存在一个 View A ， A 的高度改变会影响其他一些 View 的布局；但很不巧的是，这些依赖于 A 的高度的 View 与 A 并不在同一个 Fragment 里面（为了做动画方便）。很自然，我决定使用 Otto 来处理 A 的高度改变事件：

{% highlight java %}
viewA.addOnLayoutChangeListener(new View.OnLayoutChangeListener() {
    @Override
    public void onLayoutChange(View v, int left, int top, int right, int bottom, int oldLeft, int oldTop, int oldRight, int oldBottom) {
        // BusProvider 是 Otto 的 Bus 单例实现，
        // HeightEvent 是 Otto 分发高度的自定义事件。
        BusProvider.getInstance().post(new HeightEvent(bottom - top));
    }
}); {% endhighlight %}

同时在另一个 Fragment 里面订阅 HeightEvent 来控制相关 View 的布局：

{% highlight java %}
@Subscribe
public void onHeightEvent(HeightEvent heightEvent) {
    // 获取 View A 的高度。
    int height = heightEvent.getHeight();

    // 布局代码...
} {% endhighlight %}

看来起来很完美，逻辑上似乎也没有问题。但是真正实现起来，有的布局并没有像我想象中的那样正确，会乱掉。

后来仔细想了一下，还是与 Otto 的使用方式有关。

A 在发布事件的时候，并不会考虑到依赖它的 View 的**状态**。举例来说，存在一个依赖 A 的高度的 View B，B 在绘制过程中会产生一些自己使用的参数，接着再用这些参数结合 A 的高度来布局，想象中这个过程是线性的。假设在 B 的参数设置完全之前， A 就发布了事件，接着就会**强迫** B 执行相关的订阅方法，由此导致布局的不正确。

为了解决这个问题，你必须搞清楚每个方法之间的**执行顺序**到底是什么样的，然后写一堆控制代码。真是不优雅。

其实发布/订阅模式本身就具有不确定性，可能会打乱订阅者本身的状态。比如在上述过程中，将我们预想中的线性执行状态打乱，提前进行了布局，导致混乱。

所以下次在使用 Otto 这类的事件总线的时候，你应当考虑到状态相关的问题。话说起来，我们在设置一些 Listener 的时候也应当考虑到状态，否则同样会导致混乱。

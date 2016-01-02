---
title: 如何控制背景圆角的显示位置？
---

Android 官方提供的 CardView 虽然给开发带来了不少便利，但是要实现类似 [V2EX](http://v2ex.com/ "V2EX") 手机版的效果，就无能为力了：

![V2EX.png](/assets/img/2016-01-02-V2EX.png "V2EX.png")

原因在于 CardView 只是简单使用了 `Canvas.drawRoundRect()` 方法绘制出圆角矩形，如果要自定义上/下/左/右的圆角是否显示，还得自己动手。一种解决方法是直接使用 .9 来模拟，但是 .9 的效果给人的感觉总是不那么醇厚，没有质感。

CardView 本质上是一个 FrameLayout ，背景的阴影效果其实是通过设置它自己扩展实现的 Drawable 来实现的。 CardView 分别实现了 RoundRectDrawable 和 RoundRectDrawableWithShadow ，前者用于 API 21 以上，而后者则承担向下兼容的任务。我们可以把这两个类直接拷贝出来，**继承**实现我们想要的效果，当然由于父类来自 CardView ，所以最终的视觉效果肯定和 CardView 差别不大。

控制圆角显示位置的原理说起来也很简单，只需要在 `Canvas.drawRoundRect()` 之后，在不需要显示圆角的地方，通过 `Canvas.drawRect()` 绘制一个小矩形将其遮盖，看起来就是没有圆角了：

{% highlight java %}
...

@Override
public void draw(Canvas canvas) {
    canvas.drawRoundRect(mBoundsF, mRadius, mRadius, mPaint);
    
    // 在 canvas 左上角对应位置绘制矩形
    canvas.drawRect(buildLeftTopRect(), mPaint);        
}

... {% endhighlight %}

接下来就是绘制阴影的问题。 Android Lollipop 本身可以通过 View 的 `setElevation()` 方法来投影，但投影出来的形状并不是说 View 是什么形状就是什么形状的，你需要指定 [Outline](http://developer.android.com/training/material/shadows-clipping.html "Defining Shadows and Clipping Views") 。 Outline 本身支持圆形、圆角矩形、矩形，当然对于我们的需求来说， 比如只显示一个圆角，其他三个都是直角的情况，肯定不会是上述三种形状中的任何一种，这时候你就需要手动绘制 Outline 所需的 Path 了，并且必须保证你绘制的图形是**凸图形**，可以通过 `Path.isConvex()` 来判断：

{% highlight java %}
...

@Override
public void getOutline(Outline outline) {
    // 判断自己绘制的 Path 是否是凸图形
    if (buildConvexPath().isConvex()) {
        outline.setConvexPath(buildConvexPath());
    } else {
        super.getOutline(outline);
    }
}

... {% endhighlight %}

对于 Lollipop 之前的系统，阴影则需要自己绘制了。不过好在 RoundRectDrawableWithShadow 本身已经有 `drawShadow()` 这个方法，并且创建有现成的阴影 Paint ，你只需要负责在 Canvas 上绘画即可，具体流程在这里不予赘述。 

于是我们就可以实现像之前提到的 V2EX 手机版的样式效果了：

![SliceDemo.png](/assets/img/2016-01-02-SliceDemo.png "SliceDemo.png")

最后附上项目地址： [mthli/Slice](https://github.com/mthli/Slice "mthli/Slice")

欢迎 Star 和 PR 。

其实这个实现我想了一下，还可以给各种圆角矩形、矩形的控件添加阴影效果，真的是很方便呢～
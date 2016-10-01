---
title: DataBinding 最强技能
---

如果这个功能不能吸引你，那么恐怕没有什么能说服你使用 [DataBinding](https://developer.android.com/topic/libraries/data-binding/index.html?hl=zh-cn "Data Binding Library") 了。

Google 推出 DataBinding ，在我看来不是什么 MVVM 之类，更重要的目的是，**增强 XML 对 View 的表现力**。在 DataBinding 推出之前，我们都习惯于使用自定义 View 来实现自己的需求，但我们认真审视一下，很多自定义 View 仅仅是简单地继承标准控件再扩展一些 XML 属性而已。这就是 XML 对 View 表现力不足的体现。

所以为什么我们扩展 XML 属性就要写自定义 View 呢？实际上我们真正需要的只是对应 XML 属性的实现方法对吧。所以 DataBinding 瞄准了这一点，推出了「根据民意调查，史上最酷的 Android 功能」，**绑定适配器 BindingAdapter** 。

BindingAdapter 只做一件事，将 XML 中定义的属性值与对应的实现方法绑定在一起。举个例子，将一个图片 URL 加载到 ImageView 中，在这里我们使用 `android:src` 属性来描述：

{% highlight xml %}
<ImageView ... android:src="@{model.url}" /> {% endhighlight %}

按照标准的 ImageView 实现， `android:src` 是不能使用 String 作为属性值的，这时候你可以考虑开始写一个自定义 View 了，或者干脆直接在你的 Activity/Fragment 里把图片手动设置到 ImageView 里。如果你使用 BindingAdpater ，那就会是这样的：

{% highlight java %}
// 这段代码你可以写在任意文件中
@BindingAdapter("android:src")
public static void setImageUrl(ImageView view, String url) {
    Picasso.with(view.getContext()).load(url).into(view);
} {% endhighlight %}

接着在编译的时候， DataBinding 就会把 `android:src` 与 `setImageUrl()` 方法绑定起来，当 `android:src` 对应的属性值为 String 时，就会调用 `setImageUrl()` 进行设置。

当然你可能说这没啥，要是能绑定自定义的属性就好了。哈哈，当然可以，现在我们不使用 `android:src` ，而是在 `attrs.xml` 里定义这样一段属性：

{% highlight xml %}
<!-- 图片 URL -->
<attr name="url" format="reference|string" /> {% endhighlight %}

接着你在 XML 布局文件中这样定义 ImageView ：

{% highlight xml %}
<ImageView ... app:url="@{model.url}" /> {% endhighlight %}

那么使用 BindingAdapter 就可以这样写：

{% highlight java %}
// 注意 app:url 与 android:src 的命名空间区别
@BindingAdapter("url")
public static void setImageUrl(ImageView view, String url) {
    Picasso.with(view.getContext()).load(url).into(view);
} {% endhighlight %}

如果你的 ImageView 有多个 XML 属性依赖，比如我希望在 ImageView 加载图片完成之前有一个占位视图：

{% highlight xml %}
<!-- 图片 URL -->
<attr name="url" format="reference|string" />

<!-- 占位图 -->
<attr name="placeHolder" format="reference" /> {% endhighlight %}

在 XML 中这样使用 ImageView ：

{% highlight xml %}
<ImageView ... 
    app:url="@{model.url}"
    app:placeHolder="@{R.drawable.place_holder}" /> {% endhighlight %}

你可以这样写 BindingAdapter ：

{% highlight java %}
// requireAll = false 表示不是所有参数都是必须的
@BindingAdapter(value = {"url", "placeHolder"}, requireAll = false)
public static void setImageUrl(ImageView view, String url, int placeHolder) {
    RequestCreator creator = Picasso.with(view.getContext()).load(url);
    if (placeHolder != 0) {
        creator.placeholder(placeHolder);
    }
    creator.into(view);
} {% endhighlight %}

实际上从上面的举例我们可以看出， BindingAdapter 不仅可以对原有 XML 属性值进行重写，还可以很方便地添加新的 XML 属性及其实现。想要 View 根据 XML 属性值做动画？当然也没有问题。 Wow , such BindingAdapter, very excited!

更多 DataBinding 的高级用法比如依赖注入、事件处理，可以参考[这篇文章](https://realm.io/cn/news/data-binding-android-boyar-mount/ "棉花糖给 Android 带来的 Data Bindings （数据绑定库）")。
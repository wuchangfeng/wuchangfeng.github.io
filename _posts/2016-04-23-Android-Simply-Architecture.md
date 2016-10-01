---
title: 一种朴素的 Android 构架分层思想 
---

关于 Android 上采用什么样的构架，印象里比较热的也就是这一年多的事。

Google 主推的 [DataBinding](https://developer.android.com/topic/libraries/data-binding/index.html?hl=zh-cn "Data Binding Library") 带起了 Android 上使用 [MVVM](https://objccn.io/issue-13-1/ "MVVM 介绍") 的浪潮。对于 DataBinding ，说实话很长一段时间我只是单纯地在模仿官方的写法，而且还写得不怎么样。由于 DataBinding 中的 Binding 对象允许你直接访问 View ，比如 `mBinding.textView` 这样，所以写了很多还是 [MVC](https://zh.wikipedia.org/wiki/MVC "MVC") 的代码，只是少了 `findViewById()` 这样的操作。另外，我由衷感到 DataBinding 只是一个包含 MVVM 思想的库，这就是说， DataBinding 只是众多 Android MVVM 实现方式中的一种，当然还有其他的实现方式；它当然还包含了很多[其他的功能](https://realm.io/cn/news/data-binding-android-boyar-mount/ "棉花糖给 Android 带来的 Data Bindings （数据绑定库）")，都是为了提升 XML 表现 View 的能力。

现阶段的我不得不承认这样一个事实， MVC 更加符合直觉。直接在作为 Controller 层的 Activity/Fragment 中写代码，尽管会变得臃肿，但紧耦合思路更清晰。其实只要适当地组织编码形式，就没问题了。当然前面这句话是废话，难道 MVVM 不是编码形式的一种吗？

既然提到 MVVM 就不能不说 [Vue](http://vuejs.org.cn/ "Vue.js") ，尽管 Vue 是 Web 端的东西，但依然给我提供了不少启发。让我们来看看 Vue 的基本原理图：

![Vue](/assets/img/2016-04-23-Vue.png "Vue")

对应到 Android 中通常是这样：

 - View -> View/ViewGroup 等。

 - Model -> JSON 对象等。

那么 ViewModel 呢？我们注意到图中的 ViewModel 包含两类事物：

 - DOM Listeners ，对应 Android 中的 `View.onClickListener()` 等。 ViewModel 负责将这些事件产生的数据变化设置到 Model 中。

 - Data Bindings ，其实就是在 Model 发生变化时通知 View 作出响应。

说白了其实就是双向绑定。我们想想， Android 中干双向绑定最合适的组件是什么？再考虑到 Android 的生命周期机制， Activity/Fragment 恐怕是最合适不过的了。可是如果把 Activity/Fragment 作为 ViewModel ，和 Activity/Fragment 作为 Controller 有什么区别？

这就是编程技巧的问题了。接下来仅仅以 Activity 举例， Fragment 同理。

来看 Activity 的 `onCreate()` 方法：

{% highlight java %}
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.main);
    // ...
} {% endhighlight %}

`R.layout.main` 假设是这样的：

{% highlight xml %}
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
             android:layout_width="match_parent"
             android:layout_height="match_parent">

    <TextView android:id="@+id/text"
              android:layout_width="match_parent"
              android:layout_height="match_parent">
    </TextView>

</FrameLayout> {% endhighlight %}

通常来说，接下来我们就会在 Activity 里使用 `findViewById()` 来获得 TextView 并进行相关的操作，标准的 Controller 逻辑。但这样的代码多起来，就是 Jake 大神的 [Butter Knife](https://github.com/JakeWharton/butterknife "JakeWharton/butterknife") 也救不了你。所以为什么这样的逻辑要放在 Activity 里面呢？ Activity 之所以会臃肿，是因为在这种情况下，它不只是 Controller ，还干涉了过多 View 层的操作，使得它也可以被归类为 View 。

为了解决这个问题，我们可以尝试尽量把 View 层的操作从 Activity 中抽离出来，放在哪里最合适呢？当然就是 `R.layout.main` 的根 View。**使用根 View 来管理所有的子 View** 。如果子 View 有需要设置数据的方法，那么就在根 View 里提供相应的外部方法，尽管这个外部方法的实现只是直接调用子 View 的设置方法；如果子 View 里有事件需要响应，就由根 View 来响应，同时提供相应的外部接口。这样， Activity 中只需要操作根 View ，实现根 View 提供的接口，而不需要关心子 View 。

还是刚才的例子，只不过根 View 换成我们自己的 CustomFrameLayout ：

{% highlight xml %}
<CustomFrameLayout
        xmlns:android="http://schemas.android.com/apk/res/android"
        android:layout_width="match_parent"
        android:layout_height="match_parent">

    <TextView android:id="@+id/text"
              android:layout_width="match_parent"
              android:layout_height="match_parent">
    </TextView>

</CustomFrameLayout> {% endhighlight %}

假设现在 Activity 需要能设置 TextView 的文字，同时能获取到 TextView 的点击事件，那么 CustomFrameLayout 就可以这样写：

{% highlight java %}
public class CustomFrameLayout extends FrameLayout {
    public interface OnTextViewClickListener {
        void onTextViewClick();
    }

    private TextView mTextView;
    private OnTextViewClickListener mListener;

    @Override
    protected void onFinishInflate() {
        super.onFinishInflate();

        mTextView = (TextView) findViewById(R.id.text);
        mTextView.setOnClickListener(new OnClickListener() {
            @Override
	    public void onClick(View view) {
                if (mListener != null) {
                    mListener.onTextViewClick();
                }
            }
        });
    }

    public void setText(CharSequence text) {
        mTextView.setText(text);
    }

    public void setListener(OnTextViewClickListener listener) {
        this.mListener = listener;
    }
} {% endhighlight %}

现在 Activity 只需要调用根 View ，也就是 CustomFrameLayout 的 `setText()` 方法即可设置 TextView 的文字；实现 `OnTextViewClickListener` 即可响应 TextView 的点击事件。

这样的分层思想用在包含大量子 View 的结构中依然不落下风，网络操作数据库操作什么的全部放在 Activity/Fragment 里也结构很清晰，毕竟 Activity/Fragment 操作的对象只有一个根 View 。你想在 View 层里写多么紧耦合的代码都无所谓。当然如果有必要的话，你还可以对子 View 进行多次分层，同时辅以自定义 View 。

那么这算 MVVM 吗？不知道，不过我自己写的挺爽的。
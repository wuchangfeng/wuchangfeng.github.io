---
title: 使用 ReactiveX 构建 MVVM
---

最近在思考一些问题。

在 Android 开发中，一个 View 的行为可能是多变的。一个稍微复杂的界面，通常都是数个 View 的行为同时在改变，比如 View A 要做偏移， View B 要做 Alpha ， View C 要隐藏等等。所以我们就会在 Activity/Fragment 里写一堆方法来维护这些 View 的状态，相当于在写一个[有限状态机](https://zh.wikipedia.org/zh-cn/%E6%9C%89%E9%99%90%E7%8A%B6%E6%80%81%E6%9C%BA "有限状态机")。

另一方面，我们拿到的数据 Model 通常是 JSON 对象，我们需要把这些对象包含的信息设置到对应的 View 上。比如需要把一个 String 设置到 TextView 里，我们会调用 TextView.setText() 方法。一个稍微复杂的界面，这样需要设置的 View 也会有多个，所以你还是会写一堆类似 Setter 的方法。数据也可以理解为一种状态，所以你还是在写状态机。

[MVC](https://zh.wikipedia.org/wiki/MVC "MVC") 随着项目增大而变得越来越复杂，与其说是因为 Controller 越来越臃肿，倒不如说是因为你需要维护的状态越来越复杂，最后脑袋就秀逗了。所以为什么我们不把 Controller 需要控制的状态切割呢？更何况很多状态本身就是 View 自己的行为，为什么我们需要从外部去改变 View 的状态，而不是让 View 自己表现出来呢？想到这里其实就很简单了，我们把 Controller 的状态切割，分散到一个个自定义 View 里， View 只维护自己的状态。维护一个小而独立的状态机总是比维护一个大而复杂的状态机要来得容易。

现在我们需要一个来转发状态的东西，很自然想到了 Bus ，或者说事件总线。同时我们还需要能够将 Model 的数据映射到 View 可以呈现的状态上，将 View 的事件映射成 Model 可以承载的数据格式（双向绑定）。也就是说 Bus 不光只做状态转发，我还希望它能在转发的过程中对状态做一些处理。那么做这样事情的最佳选择，显然就是 [ReactiveX](http://reactivex.io/ "ReactiveX") 了。

![Jake Wharton's Twitter](/assets/img/2016-04-09-Jake.png "Jake Wharton's Twitter")

其实讲了半天，只是把 MVVM 的思想又复述了一遍而已。只是我很好奇的是，为什么 Google 了一下， Android 关于 MVVM 的实践文章大多集中在 [DataBinding](http://developer.android.com/intl/zh-cn/tools/data-binding/guide.html "Data Binding Guide") 的使用上，鲜有和本文类似的观点。 DataBinding 至今官方都没有很好的实践，个人观点，写起来比较恶心，而且还没有正式版。倒是 iOS 开发中使用 ReactiveX 实现 MVVM 的资料比较多， ReactiveCocoa 更是直接支持 [ReactiveViewModel](https://github.com/ReactiveCocoa/ReactiveViewModel "ReactiveViewModel") ，更详细的文章推荐阅读这篇译文[《 MVVM 介绍》](http://objccn.io/issue-13-1/ "MVVM 介绍")。

另外对于 View 自身无法掌控的行为，比如滑动时多个 View 联动，并不是说分割的思想就不太好了。同样在 Activity/Fragment 里处理滚动事件，但是仅限于控制多个 View 的联动，不要去干涉 View 的其他行为。我们的本质思想是**分割状态**，不同层级的状态应该交给不同层级的组件去管理，不是一味地拍平，或者一味地聚集在一起。

最后附上一篇 Medium 的文章 *[RxJava: Android MVVM App structure with Retrofit](https://medium.com/@manuelvicnt/rxjava-android-mvvm-app-structure-with-retrofit-a5605fa32c00#.9nllsmr4b "RxJava: Android MVVM App structure with Retrofit")* 。其实我们不只可以使用 DataBinding 来做 MVVM 。可以不使用 [MVP](https://zh.wikipedia.org/wiki/Model_View_Presenter "MVP") 。

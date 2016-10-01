---
title: 简单提升编译速度的一个方法
---

随着项目越来越大，编译速度越来越是一个问题。在编译我们的 Android App 的时候，印象里最快的时候也得一分半，当然这还是在关闭 Chrome 的时候。当你改几行代码时，仍然要花上几分钟来编译，这是很操蛋的一件事。Google 官方推出的 [Instant Run](https://developer.android.com/studio/run/index.html#instant-run "Instant Run") 看上去很美好，但也是很操蛋的，广大人民群众纷纷表示，为什么我改了代码以后，编译不生效啊摔（

增加电脑内存是提升编译速度的一种方法，Gradle 官方推荐的编译内存为 5120MB ：

![5120MB](/assets/img/2016-07-07-5120MB.png "5120MB")

当然大多数开发者的电脑可能没有这么高的内存（包括我），尤其是电脑同时运行着 AS 和 Chrome 的情况下。 不过值得庆幸的是，Google 官方在[介绍 MultiDex 的文档](https://developer.android.com/studio/build/multidex.html#dev-build "Optimizing Multidex Development Builds")中提到，通过限定 minSdkVersion 为 21 可以提升编译速度，原因在于省略了 MultiDex 向后兼容的过程，而这通常是编译最耗时的部分。在项目的 `build.gradle` 中按照如下配置：

{% highlight groovy %}
android {
    productFlavors {
        dev {
            // 开发中使用的最低支持版本
            minSdkVersion 21
        }
        prod {
            // 项目实际的最低支持版本
            minSdkVersion 14
        }
    }
}

dependencies {
    // 提升编译速度必要的依赖（我们设置为仅在 dev 时生效）
    devCompile 'com.android.support:multidex:1.0.1'
} {% endhighlight %}

上述配置完成之后，按照 Google 的说法，我们在编译 dev 版本时，就是**增量编译**。

下图为 clean 后第一次编译时所花费的时间：

![First](/assets/img/2016-07-07-First.png "First")

将几行代码注释以后，再次编译的时间：

![Second](/assets/img/2016-07-07-Second.png "Second")

可以看到，尽管第一次编译的时间较长，但从第二次编译开始，时间缩短不少。相对于以前改几行代码都要等上几分钟的编译的时间，已经是很大的提升了。

需要注意的是，在修改 `build.gradle` 后，我们需要手动将 AS 的 Build Variants 设置为 `prod` 版本，如果是 `dev` 的话，AS 不会显示 API 21 的限制提示；使用命令行编译 `dev` 版本，而不是使用 AS 的 Run ，否则编译出来的还会是 `prod` 版本，时间并不会减少。当然对于非 MultiDex 的用户，这个方法是否生效，还没有测试过。另外此方法似乎只对 Android 6.0 及其以上版本的系统才生效，5.X 会抛出 ClassNotFoundException 。
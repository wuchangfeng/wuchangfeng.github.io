---
title: 如何实现一个 Android 端的富文本编辑器组件？
---

最近处于自己的兴趣和其他一些原因，我决定尝试实现一个简单的 Android 端富文本编辑器组件。

项目地址： [mthli/Knife](https://github.com/mthli/Knife "mthli/Knife")

考察了一翻，在 Android 上实现富文本编辑器的思路大致分为三种：

 1. 使用多种 Layout 布局，每一种布局对应一种 HTML 格式，比如图片，比如顺序列表等。具体的实现例子可以参考[这个链接](https://github.com/xmuSistone/android-animate-RichEditor "xmuSistone/android-animate-RichEditor")。 Medium 和 Evernote 的富文本编辑就是采用这种方式实现的。总体来说比较复杂。

 2. WebView + JavaScript 实现。现在 Web 端有很多成熟的 JavaScript 富文本编辑库，比如 [Squire](https://github.com/neilj/Squire "neilj/Squire") ，你只需要做好 WebView 和 JavaScript 的交互就可以了（多写回调函数）。理论上虽然是这么说，但是在实现过程你需要解决 WebView 的兼容性问题（ Android 4.4 及其以上版本和 4.4 以下版本的 WebView 内核不一样），以及其他一些不可预见的问题（比如我就遇到无法粘贴文字的问题）。

 3. EditText + Span 。 Android 的 TextView 原生支持诸如粗体、删除线、引用等 Span ，要实现简单的富文本编辑需求，可操作性还是比较大的。综合再三，我选择了这种方式来实现自己的需求。

既然决定使用 EditText + Span 的方式来实现，必然要对相关的 API 有所了解。

## Span

Span 是一个强大的概念，有兴趣深入的同学我推荐直接阅读[这篇译文](http://rocko.xyz/2015/03/04/%E3%80%90%E8%AF%91%E3%80%91Spans%EF%BC%8C%E4%B8%80%E4%B8%AA%E5%BC%BA%E5%A4%A7%E7%9A%84%E6%A6%82%E5%BF%B5/ "【译】Spans，一个强大的概念")。

在这里我们我们主要使用两种类型的 Span ：

 - 继承自 CharacterStyle 的 Span ，比如 StyleSpan ，可以在**字符级别**上添加粗体，下划线等。

 - 继承自 ParagraphStyle 的 Span ，比如 QuoteSpan ，可以为**段落级别**的文本添加引用。

接着我们需要一个可以将 Span 的效果设置进去的文本结构（即实现了 Spannable 接口）， SpannableStringBuilder 是个不错的选择，同时 EditText 提供的 `getEditableText()` 方法也可以获得。通常我们只需要 `getEditableText()` 就可以了，但是在面对一些细节部分，我们可以使用 SpannableStringBuilder 预先设置相应的 Span ，再替换到原来的文本中。

设置 Span 的方式也很简单，我们需要调用 `Spannable.setSpan(Object what, int start, int end, int flags)` 这个方法即可，方法中 4 个参数的解释如下：

 - `Object what` ，传入你使用的 Span 对象。

 - `int start` ，设置 Span 的开始位置。

 - `int end` ，设置 Span 的结束位置。

 - `int flags` ，代表设置 Span 的作用域。

在这里重点介绍一下 `int flags` 这个参数，它接受 4 种类型的参数，分别是：

 - `Spanned.SPAN_INCLUSIVE_EXCLUSIVE` ，表示你在设置 Span 的区域之前输入文字，输入的文字也会受到 Span 的影响。

 - `Spanned.SPAN_INCLUSIVE_INCLUSIVE` ，表示你在设置 Span 的区域前后输入文字，输入的文字都后受到 Span 的影响。

 - `Spanned.SPAN_EXCLUSIVE_EXCLUSIVE` ，表示你在设置 Span 的区域中出输入文字，输入的文字才会受到 Span 的影响。

 - `Spanned.SPAN_EXCLUSIVE_INCLUSIVE` ，表示你在设置 Span 的区域之后输入文字，输入的文字也会受到 Span 的影响。

「受到影响」的意思就是，仍然会保持你设置的 Span 的样式，比如我选择 `Spanned.SPAN_EXCLUSIVE_INCLUSIVE` 设置了一段文字的粗体，那么我在这段文字后新输入的文字，也会是粗体。在这里我推荐使用 `Spanned.SPAN_EXCLUSIVE_EXCLUSIVE` 参数，毕竟其他几种参数相对不是很好控制，而且会给使用的人带来的疑惑。我认为一个操作代表的行为应当是**准确没有歧义**的。

好，到这里我们已经知道大致怎么作出一个富文本编辑器组件的样子了，无非是指定开始位置和结束位置，再设置相应的 Span 即可。至于设置的时候采取什么样的规则，你可以自己定制。但我们仅仅解决了**编辑的问题**，仍然存在**导入的问题**和**导出的问题**。

## 如何导入和导出？

导入的问题十分简单， Android SDK 中提供了 `Html.fromHtml()` 这个方法，我们可以很轻松地将 HTML 字符串转换为所需的 Spanned 对象。但是需要注意的是， `Html.fromHtml()` 并不支持所有的 HTML 标签，比如无序列表就不支持，因此你需要自己实现 `Html.TagHandler` 接口来处理自己所需的标签，可以参考[这个链接](https://github.com/mthli/Knife/blob/master/knife/src/main/java/io/github/mthli/knife/KnifeTagHandler.java "Knife/knife/src/main/java/io/github/mthli/knife/KnifeTagHandler.java")，实现了删除线和简单无序列表的支持。

面对粗体、斜体这样**字符级别**的样式， `Html.fromHtml()` 会自然而然的解析，该添加换行的地方就添加换行，并没有什么问题；但是面对引用、无序列表这样**段落级别**的样式，该方法会追加一个换行，也就是两个换行操作，相当于多出一个空行。通常来说我们认为一个 \<p> 对应两个 \<br> ，但是如果你有特别需求的话，也可以通过前面说的那样，自己来解析，而不是用系统默认的方式。

之前介绍了如何导入，想必你也十分清楚，必然有一个对应的 `Html.toHtml()` 方法！没错，但是遗憾的是，这个方法也不全支持所有 Span ，比如列表就不支持。不过没有关系， `Html.toHtml()` 这个方法本身的源码简洁易懂，可以参考着实现。

在这里重点说明 Spannanle 的一个接口方法 `nextSpanTransition(int start, int limit, Class type)` ，这个方法会在你指定的文本范围内，返回下一个你指定的 Span 类型的开始位置，依照这个方法，我们就可以逐层扫描指定的 Span ，而不用同时考虑其他类型的 Span 的影响，十分有用。

最后尽管说了这么多，导入导出还是有一个比较关键的问题，即导入的内容和导出的内容要**保持一致**，在这点上目前我还比较难以实现，只能说尽量控制吧，必要的时候还需要使用一下**正则**来处理导入导出的文本。

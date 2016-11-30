---
title: Android：理解 Dagger2 中的 Scope 用法
toc: true
date: 2016-07-14 20:27:39
tags: dagger2 android
categories: About Java
description:
feature:
---

本篇记录 Dagger2 中的 scope 以及一些重要的知识点。

<!--more-->

Dagger2 中虽然概念挺多的,但是大部分花时间都能理清。包括看人家的分析,Debug 代码下去也能懂。但是对于 scope 的用法以及实现原理还是有点难理解的。主要的问题也像简书上的文章所说:

- 自定义注解是怎么工作的？是不是命名了就能达到自己想要达到能够控制自己所提供的组建的生命周期的生命周期的功能？引用的说法就是这么一个回事:

  > 比如Singleton就可以实现单例，PerActivity就可以创建的类实例与Activity**“共生死“**，是不是我定义一个PerFragment的注解，同样可以达到创建的类实例就与Fragment**“共生死“**。

- Singleton 是 scope 的默认实现。也就是说 Singleton 是 scope 的一个表现形式,系统已经给我们提供了。那么为什么需要这样一个 Singleton 呢？以及这个 SingleTon 是怎么实现出来的。

对于后面说的一点,看过很多文章去尝试理解。

一个 app 要有一个全局的 Component（我们暂且叫ApplicationComponent），ApplicationComponent 负责管理整个 app 用到的全局类实例，那不可否认的是这些全局类实例应该都是单例的。

就有人说:**Singleton** 并没有创建单例的能力。不是说你用 Singleton 修饰了某个实例,它在全局范围内就是单例的了。真正用创建 Singleton 的过程步骤为:

- 在 Module 中定义创建全局类实例的方法
- ApplicationComponent 管理 Module
- 保证 ApplicationComponent 只有一个实例（在 app 的 Application 中实例化 ApplicationComponent ）

**所以讲 Singleton 并没有创建单例的能力,进而引申出并不是用 Scope 修饰的实例就有一定的生命周期的控制等等概念。**

看到一篇文章底下的评论如是说:

> 用MyScope 标注的 Component，如果 Moudle 中的 provide 也被MyScope标注，那么在这个Component的生命周期内 ，这个 provide提供的对象是单例的。

这样一看,仔细想一下也是能够理解的。更多的时候我们的 scope 以及 Singleton 只是一个标识符而已。

最后牛晓伟的那篇文章最后一些还是没明白:

> - 更好的管理 Component 之间的组织方式，不管是**依赖方式**还是**包含方式**，都有必要用自定义的Scope 注解标注这些 Component，这些注解最好不要一样了，不一样是为了能更好的体现出Component 之间的组织方式。还有编译器检查有依赖关系或包含关系的 Component，若发现有Component 没有用自定义 Scope 注解标注，则会报错。



> - 更好的管理 Component 与 Module 之间的匹配关系，编译器会检查 Component 管理的 Modules，若发现标注 Component 的自定义 Scope 注解与 Modules 中的标注创建类实例方法的注解不一样，就会报错。
>
>
> - 可读性提高，如用 Singleton 标注全局类，这样让程序猿立马就能明白这类是全局单例类。

特别对于第一点还是不太明白。记录下来以后还是实践中多总结吧。
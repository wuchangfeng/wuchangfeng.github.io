---
title: 阅读 Okhttp3.0 以及 Retrofit2.0 相关技术博客总结
toc: true
date: 2016-05-18 21:20:55
tags: java
categories: About Java
description:
feature:
---

阅读关于 Android 通信库 OKhttp ，Retrofit 的一些博客以及源码的收获和总结。

<!--more-->

## 一 . OKHttp

阅读以下文章：

[OkHttp3.0](http://www.jianshu.com/p/92a61357164b)

**关于复用连接池**

Http 通信时，要先进行 Tcp 握手，保证通信传输的可靠性，但是频繁的建立/断开，自然效率不高，因而出来了 keep-alive 一词。浏览器跟服务器均能视不同情况保持一定数量的 keep-alive Socket。

okhttp Okhttp支持5个并发，默认链路生命为5分钟(链路空闲后，保持存活的时间)。

更进一步 okhttp 为了更好的管理这些 Socket，建立了复用连接池。判断 Socket 是否还有效，用的是 引用计数法(虽然说主流 JVM 都淘汰了这种方式)，清除无效的 Socket 用的算法是 标记-清除。另外 引用计数法采用的最弱的引用方式-虚引用，虚引用存在的作用就是为了跟踪对象被引用的状态。

另外还有 弱引用来保持一层防护，在找不到满足清除条件的连接时。找到最不活跃的连接来进行清除。

通过以上步骤就可以保存多个活跃的健康的 keep-alive 链接。

**关于缓存策略**

这个涉及的主要是 HTTP 缓存策略比较简单，可以直接参考 [OkHTTP 源码分析(缓存策略)](http://www.jianshu.com/p/9cebbbd0eeab)

**关于任务队列**

okhttp 对于任务的请求采用了 Dispatcher(反向代理)技术和线程池(ExecutorService 来创建单例线程池)相结合的方式进行管理，实现了高并发和低阻塞的运行方式。

而对于反向代理，是一种典型的单生产者多消费者的模型。这样的好处自然很多，显然减少了对于单个服务器的压力，通过 Dispatcher 去分发这些请求到不同的服务器上，实现负载均衡。

另外 okhttp 还用双端队列来进行任务的缓存。而对于等待队列的移动 okhttp 采用了在 try/finally 中调用 finished 函数，而并没有用锁来实现。

## 二 . Retrofit

阅读以下文章：

[Retrofit 源码解析](http://www.jianshu.com/p/c1a3a881a144)
[Retrofit 漂亮的解耦套路](http://www.jianshu.com/p/45cb536be2f4)
[Retrofit 下动态代理的分析](http://www.jianshu.com/p/a56c61da55dd)

**总体设计**

Retrofit 总体上设计的非常巧妙。

> Retrofit非常巧妙的用注解来描述一个HTTP请求，将一个HTTP请求抽象成一个Java接口，然后用了Java动态代理的方式，动态的将这个接口的注解“翻译”成一个HTTP请求，最后再执行这个HTTP请求

**运行机制**

> 刚才讲到GitHub github = retrofit.create(GitHub.class);代码返回了一个动态代理对象，而执行Call<List<Contributor>> call = github.contributors("square", "retrofit");代码时返回了一个OkHttpCall对象，拿到这个Call对象才能执行HTTP请求

另外在 create() 函数中有这样一句代码：

``` java
return new MethodHandler<>(retrofit.client(), requestFactory, callAdapter, responseConverter);
```

MethodHandler对象，一个MethodHandler对象中包含了4个对象

1. okhttpclient:retrofit 基于 okhttp,故这个是默认生成的。
2. requestFactory:解析一个接口，比如上面的Github接口，结果就是得到整个Http请求全部的信息，还会通过@Path和@Query注解拼接Url
3. classAdapter: 让不同平台都能使用，即已经存在的OkHttpCall，要被不同的标准，平台来调用
4. responseConverter:它将response转换成我们具体想要的T。Retrofit提供了很多converter factory。比如Gson，Jackson，xml，protobuff等等。你需要什么，就配置什么工厂。在Service方法上声明泛型具体类型就可以了。


获取以上四个对象就是为了执行下面这句：

``` java
Object invoke(Object... args) {
  		return callAdapter.adapt(new OkHttpCall<>(client, requestFactory, responseConverter, args));
}
```

这句也是动态代理里面的核心方法。最后返回一个 call 对象，接着调用Call对象的execute()或enqueue(Callback<T> callback)方法，就能发送一个Http请求了。
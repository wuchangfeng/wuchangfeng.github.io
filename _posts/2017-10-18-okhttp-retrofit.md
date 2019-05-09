---
title: 阅读 Okhttp3.0 以及 Retrofit2.0 相关技术博客总结
toc: true
date: 2017-05-18 21:20:55
tags: java
categories: About Java
description:
feature:
---

阅读关于 Android 通信库 OKhttp ，Retrofit 的一些博客以及源码的收获和总结。

<!--more-->

## 零 . Http 

阅读该篇文章：

[关于Http你需要的知道一切](https://mp.weixin.qq.com/s?__biz=MzIxNzU1Nzk3OQ==&mid=2247485056&idx=1&sn=006fb62b587ee62c7a198d37eee11d7f&chksm=97f6b834a08131221bcd03a80446e3b182da7c8448295917bf4cb8e14dde948eff2420481936&scene=0&key=baf732038d89126b86297a8b8d9be5e9415a98c6f1852ce30c197f3ca9cbac14a345afffe0b3c57e1e38e2f775457d55f3ee3c94ce384fdd776cc16f71fe65fe86bb816dafb3fa17f1336ba4f7ccefed&ascene=0&uin=OTc3MjY5NTIw&devicetype=iMac+MacBookPro12%2C1+OSX+OSX+10.12.5+build(16F73)&version=12020810&nettype=WIFI&fontScale=100&pass_ticket=hwzSKMlKYin2YY3Y76YbJ3%2BPC12mVLjdqxODq9yR1ytchtIsKpccMvqHA1%2F6ImO8)

#### Http 优化方向

带宽：如果说我们还停留在拨号上网的阶段，带宽可能会成为一个比较严重影响请求的问题，但是现在网络基础建设已经使得带宽得到极大的提升，我们不再会担心由带宽而影响网速，那么就只剩下延迟了。
延迟：

1. 浏览器阻塞（HOL blocking）：浏览器会因为一些原因阻塞请求。浏览器对于同一个域名，同时只能有 4 个连接（这个根据浏览器内核不同可能会有所差异），超过浏览器最大连接数限制，后续请求就会被阻塞。
2. DNS 查询（DNS Lookup）：浏览器需要知道目标服务器的 IP 才能建立连接。将域名解析为 IP 的这个系统就是 DNS。这个通常可以利用DNS缓存结果来达到减少这个时间的目的。
3. 建立连接（Initial connection）：HTTP 是基于 TCP 协议的，浏览器最快也要在第三次握手时才能捎带 HTTP 请求报文，达到真正的建立连接，但是这些连接无法复用会导致每次请求都经历三次握手和慢启动。三次握手在高延迟的场景下影响较明显，慢启动则对文件类大请求影响较大。


#### Http1.0 与 Http1.1 的区别

HTTP1.0最早在网页中使用是在1996年，那个时候只是使用一些较为简单的网页上和网络请求上，而HTTP1.1则在1999年才开始广泛应用于现在的各大浏览器网络请求中，同时HTTP1.1也是当前使用最为广泛的HTTP协议。 主要区别主要体现在：
1. 缓存处理，在HTTP1.0中主要使用header里的If-Modified-Since,Expires来做为缓存判断的标准，HTTP1.1则引入了更多的缓存控制策略例如Entity tag，If-Unmodified-Since, If-Match, If-None-Match等更多可供选择的缓存头来控制缓存策略。
带宽优化及网络连接的使用，HTTP1.0中，存在一些浪费带宽的现象，例如客户端只是需要某个对象的一部分，而服务器却将整个对象送过来了，并且不支持断点
2. 续传功能，HTTP1.1则在请求头引入了range头域，它允许只请求资源的某个部分，即返回码是206（Partial Content），这样就方便了开发者自由的选择以便于充分利用带宽和连接。
3. 错误通知的管理，在HTTP1.1中新增了24个错误状态响应码，如409（Conflict）表示请求的资源与资源的当前状态发生冲突；410（Gone）表示服务器上的某个资源被永久性的删除。
4. Host头处理，在HTTP1.0中认为每台服务器都绑定一个唯一的IP地址，因此，请求消息中的URL并没有传递主机名（hostname）。但随着虚拟主机技术的发展，在一台物理服务器上可以存在多个虚拟主机（Multi-homed Web Servers），并且它们共享一个IP地址。HTTP1.1的请求消息和响应消息都应支持Host头域，且请求消息中如果没有Host头域会报告一个错误（400 Bad Request）。
5. 长连接，HTTP 1.1支持长连接（PersistentConnection）和请求的流水线（Pipelining）处理，在一个TCP连接上可以传送多个HTTP请求和响应，减少了建立和关闭连接的消耗和延迟，在HTTP1.1中默认开启Connection： keep-alive，一定程度上弥补了HTTP1.0每次请求都要创建连接的缺点。

#### SPDY 对 Http1.x 的优化

2012年google如一声惊雷提出了SPDY的方案，优化了HTTP1.X的请求延迟，解决了HTTP1.X的安全性，具体如下：
1. 降低延迟，针对HTTP高延迟的问题，SPDY优雅的采取了多路复用（multiplexing）。多路复用通过多个请求stream共享一个tcp连接的方式，解决了HOL blocking的问题，降低了延迟同时提高了带宽的利用率。
2. 请求优先级（request prioritization）。多路复用带来一个新的问题是，在连接共享的基础之上有可能会导致关键请求被阻塞。SPDY允许给每个request设置优先级，这样重要的请求就会优先得到响应。比如浏览器加载首页，首页的html内容应该优先展示，之后才是各种静态资源文件，脚本文件等加载，这样可以保证用户能第一时间看到网页内容。
3. header压缩。前面提到HTTP1.x的header很多时候都是重复多余的。选择合适的压缩算法可以减小包的大小和数量。
基于HTTPS的加密协议传输，大大提高了传输数据的可靠性。
4. 服务端推送（server push），采用了SPDY的网页，例如我的网页有一个sytle.css的请求，在客户端收到sytle.css数据的同时，服务端会将sytle.js的文件推送给客户端，当客户端再次尝试获取sytle.js时就可以直接从缓存中获取到，不用再发请求了。SPDY构成图：


#### Http2.0 与 Http1.x 的区别

1. 新的二进制格式（Binary Format），HTTP1.x的解析是基于文本。基于文本协议的格式解析存在天然缺陷，文本的表现形式有多样性，要做到健壮性考虑的场景必然很多，二进制则不同，只认0和1的组合。基于这种考虑HTTP2.0的协议解析决定采用二进制格式，实现方便且健壮。
2. 多路复用（MultiPlexing），即连接共享，即每一个request都是是用作连接共享机制的。一个request对应一个id，这样一个连接上可以有多个request，每个连接的request可以随机的混杂在一起，接收方可以根据request的 id将request再归属到各自不同的服务端请求里面。
3. header压缩，如上文中所言，对前面提到过HTTP1.x的header带有大量信息，而且每次都要重复发送，HTTP2.0使用encoder来减少需要传输的header大小，通讯双方各自cache一份header fields表，既避免了重复header的传输，又减小了需要传输的大小。
4. 服务端推送（server push），同SPDY一样，HTTP2.0也具有server push功能。


## 一 . OKHttp

阅读以下文章：

[OkHttp3.0](http://www.jianshu.com/p/92a61357164b)

**OkHttp 的优势**

* 支持 HTTP2，而 HTTP2 通过多路复用技术支持在一个单独的 TCP 上进行并发
* 如果 HTTP2 不可用，连接池复用技术可以降低延时
* 支持 GZIP，压缩下载体积
* 响应缓存，可以减少请求次数
* 会从很多常用的连接问题中自动恢复
* 如果您的服务器配置了多个IP地址, 当第一个IP连接失败的时候, OkHttp会自动尝试下一个IP
* OkHttp还处理了代理服务器问题和SSL握手失败问题

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

**基本用法**

第一步，声明API接口

```java
public interface GitHubService {
  @GET("users/{user}/repos")
  Call<List<Repo>> listRepos(@Path("user") String user);
}
```

第二步，构造出 `Retrofit` 对象：

```java
Retrofit retrofit = new Retrofit.Builder()
    .baseUrl("https://api.github.com/")
    .addConverterFactory(GsonConverterFactory.create())
    .build();
```

第三步，得到 API 接口，直接调用：

```java
GitHubService service = retrofit.create(GitHubService.class);
Call<List<Repo>> repos = service.listRepos("octocat");
```

最后，就是调用 `repos` 执行 Call ：

```java
// sync
repos.execute();
// async
repos.enqueue(...);
```

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
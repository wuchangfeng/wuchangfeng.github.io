---
layout: post
title: 个人知识体系构建-系列一
date: 2017-04-14 21:58:37 +0800
categories: interview
---

个人知识体系构建，涵盖 网络、操作系统、数据结构、Java 语言、Android 开发。主要为实习和校招准备。

> 主要概览

- HTTP 协议入门
- HTTP1.0、HTTP1.1、HTTP2、HTTP 缓存、HTTP 状态码
- 断点上传/下载如何实现
- Java  new 一个对象的过程
- Glide 中缓存以及 BitmapPool 实现机制
- 图片加载框架的设计
- 数据库事物的理解以及四个特性（ACID）
- Hashset 底层实现原理
- ConcurrentHashMap 的结构以及实现原理
- Android 中进程保活机制
- Android 中 WebView 控件的使用
- Android 热修复三个主要的流派
- Android 性能优化系列
- Android 中责任链模式的实现
- Android 耗电量的计算
- 热修复相关技术整理
- Apk 的打包流程
- Binder 机制

> 具体内容

- sync 和 lock 之间的区别

  - synchronized和lock的用法区别

    - synchronized(隐式锁)：在需要同步的对象中加入此控制，synchronized可以加在方法上，也可以加在特定代码块中，括号中表示需要锁的对象。
    - lock（显示锁）：需要显示指定起始位置和终止位置。一般使用ReentrantLock类做为锁，多个线程中必须要使用一个ReentrantLock类做为对 象才能保证锁的生效。且在加锁和解锁处需要通过lock()和unlock()显示指出。所以一般会在finally块中写unlock()以防死锁。

  - synchronized和lock性能区别

    synchronized是托管给JVM执行的，而lock是java写的控制锁的代码。在Java1.5中，synchronize是性能低效的。因为 这是一个重量级操作，需要调用操作接口，导致有可能加锁消耗的系统时间比加锁以外的操作还多。相比之下使用Java提供的Lock对象，性能更高一些。但 是到了Java1.6，发生了变化。synchronize在语义上很清晰，可以进行很多优化，有适应自旋，锁消除，锁粗化，轻量级锁，偏向锁等等。导致 在Java1.6上synchronize的性能并不比Lock差。

  - synchronized和lock机制区别

    - synchronized原始采用的是CPU悲观锁机制，即线程获得的是独占锁。独占锁意味着其 他线程只能依靠阻塞来等待线程释放锁。
    - Lock用的是乐观锁方式。所谓乐观锁就是，每次不加锁而是假设没有冲突而去完成某项操作，如果因为冲突失败就重试，直到成功为止。乐观锁实现的机制就 是CAS操作（Compare and Swap）。


- RxJava 优缺点
  - 异步编程，避免陷入各种回调事件中
  - 避免代码各种蜜汁缩进
  - 项目较大忘记解除注册，导致内存泄漏
  - 使用成本问题

- 各种排序算法总结  http://www.cnblogs.com/nannanITeye/archive/2013/04/11/3013737.html

  ![](http://ww1.sinaimg.cn/large/b10d1ea5ly1feih8dnu5vj20iz07874f.jpg)


- HTTPS实现原理以及对称加密和非对称加密

  - [HTTPS 实现原理](http://qifuguang.me/2017/03/25/%E7%9C%8B%E5%AE%8C%E8%BF%98%E4%B8%8D%E6%87%82HTTPS%E6%88%91%E7%9B%B4%E6%92%AD%E5%90%83%E7%BF%94/)
  - [对称加密和非对称加密](http://qifuguang.me/2016/03/27/%E5%AF%B9%E7%A7%B0%E5%8A%A0%E5%AF%86%E4%B8%8E%E9%9D%9E%E5%AF%B9%E7%A7%B0%E5%8A%A0%E5%AF%86/)  具体的加密算法推荐阮一峰老师的 RSA 加密算法解析
  - SSL 整数的握手过程 http://luojinping.com/2016/04/17/https%E8%AF%A6%E8%A7%A3/

- HTTP 协议入门 http://www.ruanyifeng.com/blog/2016/08/http.html

  - HTTP 协议基本内容
  - HTTP1.0 、HTTP2.0
  - HTTP 请求头
  - HTTP 响应头

- TCP 如何保证可靠性

  - 应用程序被分割成 TCP 认为合适发送的数据块
  - TCP发出一段报文之后，启动一个定时器，等待目的端确认，如果没能及时收到，则重传
  - TCP 将保持它首部和数据的校验和，这是一个端到端校验和，如果收到的报文段的校验和有差错，TCP将丢弃该报文段，同时不发送确认收到的消息，从而使发送端超时重发。
  - TCP 提供流量控制

- IP 分片与 TCP 分片

  - IP 分片 MTU(最大传输单元)1500个字节
  - TCP 分片 MSS(最大分段大小) 1460个字节

- HTTP1.1 与 HTTP 1.0 之间的区别

  - 增加持久链接：就是引入了持久连接（persistent connection），即TCP连接默认不关闭，可以被多个请求复用

  - 管道机制：即在同一个TCP连接里面，客户端可以同时发送多个请求。这样就进一步改进了HTTP协议的效率。

  - Content-length 字段：一个TCP连接现在可以传送多个回应，势必就要有一种机制，区分数据包是属于哪一个回应的。这就是`Content-length`字段的作用，声明本次回应的数据长度。长度之后的数据，就属于下一次响应的内容了。

  - 分块传输编码

  - 缺点：虽然1.1版允许复用TCP连接，但是同一个TCP连接里面，所有的数据通信是按次序进行的。服务器只有处理完一个回应，才会进行下一个回应。要是前面的回应特别慢，后面就会有许多请求排队等着。这称为["队头堵塞"](https://zh.wikipedia.org/wiki/%E9%98%9F%E5%A4%B4%E9%98%BB%E5%A1%9E)（Head-of-line blocking）。

    为了避免这个问题，只有两种方法：一是减少请求数，二是同时多开持久连接。这导致了很多的网页优化技巧，比如合并脚本和样式表、将图片嵌入CSS代码、域名分片（domain sharding）等等。如果HTTP协议设计得更好一些，这些额外的工作是可以避免的。

- HTTP2.0 的特点，其实也就是与 HTTP1.1 之间的区别

  - 支持二进制协议
  - 支持多工
  - 支持数据流
  - 支持头信息压缩
  - 支持服务器推送

- HTTP 缓存

  - 请求处理

    在内存或者磁盘中获取 http 请求对应的 url

  - 新鲜度检测

    检查上述的副本 url 有没有过期

  - 服务器再验证

    本地过期，并不代表跟服务器上的不一样，所以再次跟服务器上的验证一下，看到底是否过期

  - 创建响应并返回

    我们希望缓存看起来就像是来自原始服务器一样，缓存将已缓存的服务器响应首部作为响应首部，发送给客户端


- HTTP 状态码的分类

  - 1XX   代表请求已经被接受，需要继续处理，临时的响应
  - 2XX   代表请求已成功被服务器接受、理解、并接受
  - 3XX  这类状态码代表需要客户端采取进一步的操作才能完成请求。通常，这些状态码用来重定向，后续的请求地址（重定向目标）在本次响应的Location域中指明。
  - 4XX 客户端请求可能发生错误
  - 5XX 代表服务器在处理请求过程中可能发生的错误

- HttpClient 与 HttpURLConnection 之间的区别

  - Httpclient对Session和Cookie处理的比较好，不像HttpUrlConnection还要setCookie和setSession；

    HttpClient上传文件非常方便，HttpUrlConnection需要拼装传输文件的协议（写起来比较繁琐）；

  - HttpUrlConnection支持Gzip压缩，HttpClient虽然支持，但是要代码去处理；

    HttpUrlConnection的Gzip在传输大文件分包trunk有问题，只适合小文件，但是bug现在已经修复。

  - HttpUrlConnection直接支持系统级的线程池，即打开的连接不会直接关掉，在一段时间内所有程序可共用；httpclient当然也能做到，但是不如官方系统底层支持好。

  - HttpUrlConnection直接在系统级做了缓存策略处理，加快重复请求的速度。

    HttpClient 功能更强，BUG更少，更容易控制细节

  - HttpClient是apache的开源实现，而HttpUrlConnection是安卓标准实现，安卓SDK虽然集成了HttpClient，但官方支持的却是HttpUrlConnection；

  - HttpUrlConnection直接支持GZIP压缩；HttpClient也支持，但要自己写代码处理；我们之前测试HttpUrlConnection的GZIP压缩在传大文件分包trunk时有问题，只适合小文件，不过这个BUG后来官方说已经修复了；

  - HttpUrlConnection直接支持系统级连接池，即打开的连接不会直接关闭，在一段时间内所有程序可共用；HttpClient当然也能做到，但毕竟不如官方直接系统底层支持好；

  - HttpUrlConnection直接在系统层面做了缓存策略处理，加快重复请求的速度。

- SPDY 介绍

  SPDY 是 Google 开发的基于 TCP 的应用层协议，用以最小化延迟网络、提升网络速度、优化用户体验。其并不是用以替代 HTTP，而是对其的一种增强。新协议包括数据流的多路复用，请求优先级、数据报头压缩等等。

- 介绍一下 OkHttp

  OkHttp是一个现代，快速，高效的Http client，支持HTTP/2以及SPDY（SPDY介绍网址：[https://zh.wikipedia.org/wiki/SPDY](https://zh.wikipedia.org/wiki/SPDY)，**SPDY**（发音如英语：speedy），一种[开放](https://zh.wikipedia.org/wiki/%E9%96%8B%E6%94%BE%E5%8E%9F%E5%A7%8B%E7%A2%BC)的[网络传输协议](https://zh.wikipedia.org/wiki/%E7%B6%B2%E8%B7%AF%E5%82%B3%E8%BC%B8%E5%8D%94%E5%AE%9A)，由[Google](https://zh.wikipedia.org/wiki/Google)开发），它为你做了很多的事情。

  - OKHttp是Android版Http客户端。非常高效，支持SPDY、连接池、GZIP和HTTP缓存。支持SPDY，可以合并多个到同一个主机的请求

  - OkHttp实现的诸多技术如：连接池，gziping，缓存等就知道网络相关的操作是多么复杂了。

  - OkHttp扮演着传输层的角色。
  - OkHttp使用[Okio](https://goo.gl/F7rG1V)来大大简化数据的访问与存储，[Okio](https://goo.gl/F7rG1V)是一个增强 java.io和 java.nio的库。
  - OkHttp 处理了很多网络疑难杂症：会从很多常用的连接问题中自动恢复。如果您的服务器配置了多个IP地址，当第一个IP连接失败的时候，OkHttp会自动尝试下一个IP。
  - OkHttp还处理了代理服务器问题和SSL握手失败问题。
  - OkHttp是一个Java的HTTP+SPDY客户端开发包，同时也支持Android。需要Android 2.3以上
  - OKHttp是Android版Http客户端。非常高效，支持SPDY、连接池、GZIP和 HTTP 缓存。
  - 默认情况下，OKHttp会自动处理常见的网络问题，像二次连接、SSL的握手问题。
  - 如果你的应用程序中集成了OKHttp，Retrofit默认会使用OKHttp处理其他网络层请求。
  - 从Android4.4开始HttpURLConnection的底层实现采用的是okHttp 

- 断点上传下载

  实现断点续传的前提是需要服务器记录某文件的上传进度，那么根据什么判断是不是同一个文件呢？可以利用文件内容求md5码，如果文件过大，求取md5码也是一个很长的过程，所以对于大文件，只能针对某一段数据进行计算，加上服务器对cookie用户信息的判断，得到相对唯一的key。在前端页面，需要将文件按照一定大小进行分片，一次请求只发送这一小片数据，所以我们可以同时发起多个请求。但一次同时请求的连接数不宜过多，服务器负载过重。对于文件分片操作，H5具有十分强大的File API，直接利用File对象的slice方法即可得到Blob对象。至于同时传输数据的连接数控制逻辑，就需要花点脑子思考了。前端把数据顺利得传给服务器了，服务器只需要按照数据中给的开始字节位置，与读取到的文件片段数据，写入文件即可。

- 哈希表的关键几点 https://bestswifter.com/hashtable/

  - 哈希因子
  - 自动扩容
  - 再哈希

- 四种垃圾回收器 http://www.importnew.com/23752.html

  - G1垃圾回收器
  - 串行垃圾回收器
  - 并行垃圾回收器
  - 并发标记垃圾扫描回收器

- 进程与线程的区别

  进程有独立的地址空间，一个进程崩溃之后，在保护模式下不会对其他进程产生影响，而线程知识一个进程中不同执行路径。线程有自己的堆栈和局部变量，但是线程之间没有自己的单独空间，一个线程死掉就等于进程死掉。所以多线程的程序要比多进程健壮。
  - 简而言之,一个程序至少有一个进程,一个进程至少有一个线程
  - 线程的划分尺度小于进程，使得多线程程序的并发性高。
  - 另外，进程在执行过程中拥有独立的内存单元，而多个线程共享内存，从而极大地提高了程序的运行效率。
  - 线程在执行过程中与进程还是有区别的。每个独立的线程有一个程序运行的入口、顺序执行序列和程序的出口。** 但是线程不能够独立执行，**必须依存在应用程序中，由应用程序提供多个线程执行控制。
  - 从逻辑角度来看，多线程的意义在于一个应用程序中，有多个执行部分可以同时执行。但操作系统并没有将多个线程看做多个独立的应用，来实现进程的调度和管理以及资源分配。**这就是进程和线程的重要区别。**


- Java 中声明一个对象的过程 http://www.jianshu.com/p/ebaa1a03c594

  ```Java
      Dog dog= new Dog()；
  ```
  当虚拟机执行到new指令时，它先在 **常量池**中查找“Dog”，看能否定位到Dog类的符号引用；如果能，说明这个类已经被加载到方法区了，则继续执行。如果没有，就让Class Loader先执行类的加载。然后，虚拟机开始为该对象分配内存，对象所需要的内存大小在类加载完成后就已经确定了。这时候只要在 **堆**中按需求分配空间即可。具体分配内存时有两种方式，第一种，内存绝对规整，那么只要在被占用内存和空闲内存间放置指针即可，每次分配空间时只要把指针向空闲内存空间移动相应距离即可，当某对象被GC回收后，则需要进行某些对象内存的迁移。第二种，空闲内存和非空闲内存夹杂在一起，那么就需要用一个列表来记录堆内存的使用情况，然后按需分配内存。

  对于**多线程**的情况，如何确保一个线程分配了对象内存但尚未修改内存管理指针时，其他线程又分配该块内存而覆盖的情况？有一种方法，就是让每一个线程在堆中先预分配一小块内存（TLAB本地线程分配缓冲），每个线程只在自己的内存中分配内存。但对象本身按其访问属性是可以线程共享访问的。

  内存分配到后，虚拟机将分配的内存空间都初始化为零值(不包括对象头)。实例变量按变量类型初始化相应的默认值（数值型为0，boolan为false），所以实例变量不赋初值也能使用。接着设置对象头信息，比如对象的哈希值，GC分代年龄等。从虚拟机角度，此时一个新的对象已经创建完成了。但从我们程序运行的角度，新建对象才刚刚开始，对象的构造方法还没有执行。**只有执行完构造方法**，按构造方法进行初始化后，对象才是彻底创建完成了。

  构造函数的执行还涉及到调用父类构造器，如果没有显式声明调用父类构造器，则自动添加默认构造器。

  到此，new运算符可以返回堆中这个对象的引用了。此刻，会根据dog这个变量是实例变量、局部变量或静态变量的不同将引用放在不同的地方：

  **如果dog局部变量，dog变量在栈帧的局部变量表，这个对象的引用就放在栈帧。**

  **如果dog是实例变量，dog变量在堆中，对象的引用就放在堆。**

  **如果dog是静态变量，dog变量在方法区，对象的引用就放在方法区。**

- BitmapPool 参考 Glide 中是如何实现的？ Glide 源码系列参考[学习](https://xiaodanchen.github.io/2016/08/19/%E8%B7%9F%E7%9D%80%E6%BA%90%E7%A0%81%E5%AD%A6%E8%AE%BE%E8%AE%A1%EF%BC%9AGlide%E6%A1%86%E6%9E%B6%E5%8F%8A%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90%EF%BC%88%E4%B8%80%EF%BC%89/) 

  - ActiveResourceCache：缓存正在使用的资源(注意是弱引用)
  - LruResourceCache：缓存最近使用过但是当前未使用过的资源，LRU算法
  - BitmapPool：缓存所有被释放的图片，内存复用，LRU算法

  其中注意一下：

  - LruResourceCache和ActiveResourceCache设计是为了尽可能的资源复用
  - BitmapPool的设计目的是为了尽可能的内存复用

  Glide 资源完整的循环过程如下：

  ![](http://ww1.sinaimg.cn/large/b10d1ea5ly1fe975j6l94j20ky0a9ab7.jpg)

  - 当我们需要显示某个资源时，Glide会先去查找LruResourceCache，找到了则将资源从LruResourceCache移除加入到ActiveResourceCache；
  - LruResourceCache找不到资源则查找ActiveResourceCache。
  - 如果在ActiveResourceCache也找不到合适的资源，则会根据加载策略从硬盘或者网络加载资源。
  - 获取数据后Glide会从BitmapPool中找寻合适的可供内存复用的废弃recycled bitmap（找不到则会重新创建bitmap对象），然后刷新bitmap的数据。
  - bitmap被转换封装为Resource缓存入ActiveResourceCache和Request对象中。
  - Request的target会获取resource中引用的bitmap并展示。
  - 当target的资源需要release时，resource会根据缓存策略被缓存到LruResourceCache，同时ActiveResourceCache中的弱引用会被删除。如果，该资源不能缓存到LruResourceCache，则资源将被recycle到BitmapPool。
  - 当需要回收内存时（比如系统内存不足或者生命周期结束），LruResourceCache将根据LRU算法recycle一些resource到BitmapPool。
  - BitmapPool会根据缓存池的尺寸和recycled resource的缓存策略来缓存resource的bitmap。
  - BitmapPool会根据LRU算法和缓存池的尺寸来释放一些老旧资源。
  - 当系统GC时，则会回收可回收的资源释放内存

- 设计一个图片加载框架的六大步骤

  - 图片的压缩
  - 图片的异步加载
  - 图片的同步加载
  - 图片硬盘缓存
  - 图片的内存缓存
  - 图片的网络拉取

- 图片加载框架的线程池设计

  核心线程数目为当前设备 CPU 核心数加1，最大容量为CPU 核心数的 2 倍加1，线程闲置时间超时时长 10s。线程池内部有一个 LinkedBlockingQueue 队列来存储线程。注意这个数据结构的特性。

- 设计模式中策略模式和策略模式的实践

- 数据库中事物是什么意思？

  事务(Transaction)，一般是指要做的或所做的事情。在计算机术语中是指访问并可能更新数据库中各种数据项的一个程序执行单元(unit)。在计算机术语中，事务通常就是指数据库事务。


- 数据库中事物应该具备的四个特性

  - 原子性 

    事务作为一个整体被执行，包含在其中的对数据库的操作要么全部被执行，要么都不执行。

  - 一致性

    事务应确保数据库的状态从一个一致状态转变为另一个一致状态。一致状态的含义是数据库中的数据应满足完整性约束。

  - 隔离性

    多个事务并发执行时，一个事务的执行不应影响其他事务的执行。

  - 持久性

    一个事务一旦提交，他对数据库的修改应该永久保存在数据库中。

- Hashset 底层实现原理 http://wiki.jikexueyuan.com/project/java-collection/hashset.html

  问题简化一点就是如何确保元素不重复？

  如果此 set 中尚未包含指定元素，则添加指定元素。更确切地讲，如果此 set 没有包含满足(e==null ? e2==null : e.equals(e2)) 的元素 e2，则向此 set 添加指定的元素 e。如果此 set 已包含该元素，则该调用不更改 set 并返回 false。但底层实际将将该元素作为 key 放入 HashMap。思考一下为什么？

  由于 HashMap 的 put() 方法添加 key-value 对时，当新放入 HashMap 的 Entry 中 key 与集合中原有 Entry 的 key 相同（hashCode()返回值相等，通过 equals 比较也返回 true），新添加的 Entry 的 value 会将覆盖原来 Entry 的 value（HashSet 中的 value 都是`PRESENT`），但 key 不会有任何改变，因此如果向 HashSet 中添加一个已经存在的元素时，新添加的集合元素将不会被放入 HashMap中，原来的元素也不会有任何改变，这也就满足了 Set 中元素不重复的特性。

- ConcurentHashMap 内存结构  http://wiki.jikexueyuan.com/project/java-collection/concurrenthashmap.html

  ConcurrentHashMap 的成员变量中，包含了一个 Segment 的数组（`final Segment[] segments;`），而 Segment 是 ConcurrentHashMap 的内部类，然后在 Segment 这个类中，包含了一个 HashEntry 的数组（`transient volatile HashEntry[] table;`）。而 HashEntry 也是 ConcurrentHashMap 的内部类。HashEntry 中，包含了 key 和 value 以及 next 指针（类似于 HashMap 中 Entry），所以 HashEntry 可以构成一个链表。所以通俗的讲，ConcurrentHashMap 数据结构为一个 Segment 数组，Segment 的数据结构为 HashEntry 的数组，而 HashEntry 存的是我们的键值对，可以构成链表。ConcurrentHashMap 的结构中包含的 Segment 的数组，在默认的并发级别会创建包含 16 个 Segment 对象的数组。通过我们上面的知识，我们知道每个 Segment 又包含若干个散列表的桶，每个桶是由 HashEntry 链接起来的一个链表。如果 key 能够均匀散列，每个 Segment 大约守护整个散列表桶总数的 1/16。

  ConcurrentHashMap 的高并发性主要来自于三个方面：

  - 用分离锁实现多个线程间的更深层次的共享访问。
  - 用 HashEntery 对象的不变性来降低执行读操作的线程在遍历链表期间对加锁的需求。
  - 通过对同一个 Volatile 变量的写 / 读访问，协调不同线程间读 / 写操作的内存可见性。


  ![](http://ww1.sinaimg.cn/large/b10d1ea5ly1femil6qakzj20dy0ext9l.jpg)


- 路由框架支持热修复跳转吗？组件化、模块化开发 具体参考这篇文章 http://www.zhimengzhe.com/Androidkaifa/208300.html

  Android系统已经给我们提供了api来做页面跳转，比如`startActivity`，为什么还需要路由框架呢？我们来简单分析下路由框架存在的意义：在一些复杂的业务场景下（比如电商），灵活性比较强，很多功能都是运营人员动态配置的，比如下发一个活动页面，我们事先并不知道具体的目标页面，但如果事先做了约定，提前做好页面映射，便可以自由配置。随着业务量的增长，客户端必然随之膨胀，开发人员的工作量越来越大，比如64K问题，比如协作开发问题。App一般都会走向组件化、插件化的道路，而组件化、插件化的前提就是解耦，那么我们首先要做的就是解耦页面之间的依赖关系。简化代码。数行跳转代码精简成一行代码。

  - 路由管理策略
  - 路由跳转
  - 路由表的数据结构实现

- 平衡二叉树的插入删除机制，以及对应的时间复杂度

  在计算机科学中，AVL树是最先发明的**自平衡二叉查找树**。在AVL树中任何节点的两个子树的高度最大差别为1，所以它也被称为**高度平衡树**。查找、插入和删除在平均和最坏情况下都是O（log n）。增加和删除可能需要通过一次或多次树旋转来重新平衡这个树。


- 进程保活得四种方式，像素点保活 https://segmentfault.com/a/1190000006251859

  进程的优先级主要有以下这几种

  - 前台进程(Foreground process)

  - 可见进程(Visible process)

  - 服务进程(Service process)

  - 后台进程(Background process)

  - 空进程(Empty process)

    进程拉活的集中方式

  - 先提升进程的优先级
  - 利用系统广播进行拉活
  - 利用第三方应用广播拉活
  - 利用 Service 机制拉活
  - Native 拉活（5.0之后失效了）
  - 利用账号同步机制
  - 利用推送进行拉活

  最后再推荐这篇文章 http://www.infoq.com/cn/articles/wechat-android-background-keep-alive

- 内存抖动是什么意思？

  页面的频繁更换，导致整个系统效率急剧下降，这个现象称为内存抖动。抖动一般是内存分配算法不好，内存太小引或者程序的算法不佳引起的页面频繁从内存调入调出。

- Android 应用中出现了内存抖动过程吗？

  具体参考文章1：http://www.xiufm.com/blog-1-520.html   文章2：http://hukai.me/android-performance-memory/

- View 的懒加载是什么意思

  其实就是延迟加载，类似于 ViewStep 那样，等到需要的时候才进行加载，进行资源的节约。还有 Fragment 的赖加载。

- startActivity 启动流程

  在Android中，我们去调用startActivit()来启动一个Activity，经过复杂的代码跳转后，最终是由ActivityManagerService（简称ams，下同）来真正的打开一个应用的Activity的。由于ams运行在system_server进程中，所以普通的app进程与ams进行通信，就涉及到跨进程通信的问题，这种情况下的跨进程，Android采用的Binder机制，通过AIDL来实现，如果对这个不了解，可以参考[前一篇关于AIDL的博文](http://my.oschina.net/liucundong/blog/649490)，如果你对这个了解了，下面我们将要分析到的代码逻辑，你会觉得似曾相识。在startActivit()的整个过程，可以分为两部分来理解：

  - app进程去调用位于system_server进程中的ams

  startActivit()会调用ams去启动app的入口Activity，如果ams发现这个app进程未启动，则会由于zygote进程fork出一个新的进程出来，然后在启动这个 app进程的ActivityThread main()方法。

  - 位于system_server进程中的ams去调用app进程

  新启动的app进程，在其ActivityThread main()方法种，会将自己和 system_server进程中的ams绑定在一起，这样ams就会在特定的时机去回调app进程中Activity的生命周期方法。

- webView 的解释 参考链接 http://www.jianshu.com/p/3fcf8ba18d7f

  直接继承自 AbsoluteLayout  。JavaScript在WebView中默认情况下是被禁用的。你可以通过附加在WebView上的WebSettings启用它。即使用getSettings()获取WebSettings，然后启用使用setJavaScriptEnabled()方法启用JavaScript。核心的是实现一个 JavacalHtml 方法。当你的WebView重载URL加载的时，WebView会自动累加访问过的网页的历史记录。您可以通过goBack()和 goForward()方法向后、向前浏览。WebViewClient就是帮助WebView处理各种通知、请求事件的。打开网页时不调用系统浏览器， 而是在本WebView中显示。

- Android 开发插件化技术主要三个流派 http://www.infoq.com/cn/articles/android-plug-ins-from-entry-to-give-up

  - 第一种是动态替换，也就是Hook。可以在不同层次进行Hook，从而动态替换也细分为若干小流派。可以直接在Activity里做Hook，重写getAsset的几个方法，从而使用自己的ResourceManager和AssetPath；也可以在更抽象的层面，也就是在startActivity方法的位置做Hook，涉及的类包括ActivityThread、Instrumentation等；最高层次则是在AMS上做修改，也就是张勇的解决方案，这里需要修改的类非常多，AMS、PMS等都需要改动。总之，在越抽象的层次上做Hook，需要做的改动就越大，但好处就是更加灵活了。没有哪一个方法更好，一切看你自己的选择。
  - 第二种是静态代理，这是任玉刚的框架采取的思路。写一个PluginActivity继承自Activity基类，把Activity基类里面涉及生命周期的方法全都重写一遍，插件中的Activity是没有生命周期的，所以要让插件中的Activity都继承自PluginActivity，这样就有生命周期了。这里主要涉及三个点：1：资源的访问  2：Activity 生命周期的管理 3：插件 ClassLoader 的管理[即管理各个插件的 DexclassLoader]
  - 第三种是Dex合并，Dex合并就是Android热修复的思想。刚才说到了两个项目——AndFix和Nuwa，它们的思想是相同的。原生Apk自带的Dex是通过PathClassLoader来加载的，而插件Dex则是通过DexClassLoader来加载的。但有一个顺序问题，是由Davlik的机制决定的，如果宿主Dex和插件Dex都有一个相同命名空间的类的方法，那么先加载哪个Dex，哪个Dex中的这个类的方法将会占山为王，后面其他同名方法都替换了。所以，AndFix热修复就是优先加载插件包中的Dex，从而实现热修复。由于热修复的插件包通常只包括一个类的方法，体量很小，和正常的插件不是一个数量级的，所以只称为热修复补丁包，而不是插件。

- 对于享元模式的理解？能够带来什么好处？

- 责任链模式在 Android 源码中的实现

  一个请求沿着一个责任链一直传递，直到该链上某个处理着处理它为止。简单来讲就是 if-else 和 switch 那种模式，某个条件满足了，就进入去处理。ViewGroup事件投递的递归调用就类似于一条责任链，一旦其寻找到责任者，那么将由责任者持有并消费掉该次事件，具体的体现在View的onTouchEvent方法中返回值的设置（这里介于篇幅就不具体介绍ViewGroup对事件的处理了），如果onTouchEvent返回false那么意味着当前View不会是该次事件的责任人将不会对其持有，如果为true则相反，此时View会持有该事件并不再向外传递。

- Android 性能优化电量监控 http://hukai.me/android-performance-battery/

  - 大数据量的传输。

  - 不停的在网络间切换。

  - 解析大量的文本数据。

   并提出了相关的优化建议：

  - 在需要网络连接的程序中，首先检查网络连接是否正常，如果没有网络连接，那么就不需要执行相应的程序。
  - 使用效率高的数据格式和解析方法，推荐使用JSON和Protobuf。
  - 目在进行大数据量下载时，尽量使用GZIP方式下载。
  - 其它：回收java对象，特别是较大的java对像，使用reset方法；对定位要求不是太高的话尽量不要使用GPS定位，可能使用wifi和移动网络cell定位即可；尽量不要使用浮点运算；获取屏幕尺寸等信息可以使用缓存技术，不需要进行多次请求；使用AlarmManager来定时启动服务替代使用sleep方式的定时任务。


- 分段和分页管理的区别

  - 页时的逻辑地址是连续的，段式的逻辑地址是二维的
  - 页式的地址是一维，而段式是二维的
  - 分页是操作系统进行的，分段是用户确定的
  - 各页可以分散在主存，每段必须占用连续的主存空间

- IPC 进程间通信分类

  - 消息传递(管道、FIFO、消息队列)
  - 同步(互斥量、条件变量、读写锁、文件记录锁)
  - 共享内存
  - 远程过程调用

- ANR 发生的原因以及如何避免

  - 运行在主线程里的任何方法都尽可能少做事情。特别是，Activity应该在它的关键生命周期方法（如onCreate()和onResume()）里尽可能少的去做创建操作。（可以采用重新开启子线程的方式，然后使用Handler+Message的方式做一些操作，比如更新主线程中的ui等）
  - 应用程序应该避免在BroadcastReceiver里做耗时的操作或计算。但不再是在子线程里做这些任务（因为 BroadcastReceiver的生命周期短），替代的是，如果响应Intent广播需要执行一个耗时的动作的话，应用程序应该启动一个 Service。（此处需要注意的是可以在广播接受者中启动Service，但是却不可以在Service中启动broadcasereciver,关于原因后续会有介绍，此处不是本文重点）
  - 避免在Intent Receiver里启动一个Activity，因为它会创建一个新的画面，并从当前用户正在运行的程序上抢夺焦点。如果你的应用程序在响应Intent广 播时需要向用户展示什么，你应该使用Notification Manager来实现。
  - 通常100到200毫秒就会让人察觉程序反应慢，为了更加提升响应,如果程序正在后台处理用户的输入，建议使用让用户得知进度，比如使用ProgressBar控件，程序启动时可以选择加上欢迎界面，避免让用户察觉卡顿，使用Systrace和TraceView找出影响响应的问题。

- Android 中解析 xml 几种方式？

  XML解析主要有三种方式，SAX、DOM、PULL。常规在PC上开发我们使用Dom相对轻松些，但一些性能敏感的数据库或手机上还是主要采用SAX方式，SAX读取是单向的，优点:不占内存空间、解析属性方便，但缺点就是对于套嵌多个分支来说处理不是很方便。而DOM方式会把整个XML文件加载到内存中去，这里Android开发网提醒大家该方法在查找方面可以和XPath很好的结合如果数据量不是很大推荐使用，而PULL常常用在J2ME对于节点处理比较好，类似SAX方式，同样很节省内存，在J2ME中我们经常使用的KXML库来解析。 

- Apk 的打包流程

  - 利用资源打包工具 aapt 打包生成 R.java 文件
  - 处理 aidl 文件，生成相应的 Java 文件
  - 编译项目源代码,生成 class 文件
  - 转换所有的 class 文件，生成 classes.dex 文件，主要还是利用的 dx 工具
  - 打包生成 apk 文件
  - 对 apk 文件进行签名
  - 对签名后的apk文件进行对其处理

- 讲解 OkHttp 和 Retrofit 最好的一篇文章 http://www.infocool.net/kb/Android/201611/215768.html

  - 支持 HTTP2、SPDY
  - 自动选择最好的线路，并支持自动重连
  - 拥有自动维护的 socket 线程池，轻松并发
  - 拥有拦截器处理请求与响应
  - 基于 headers 的缓存策略管理

- ThreadLocal 实现原理
  ThreadLocal如何为每个线程创建变量的副本的步骤已经明了。首先，在每个线程Thread内部有一个ThreadLocal.ThreadLocalMap类型的成员变量threadLocals，这个threadLocals就是用来存储实际的变量副本的，内部由名为table的Entry数组维护，Entry的键值为当前ThreadLocal变量，value为变量副本（即T类型的变量）。初始时，在Thread里面，threadLocals为空，当通过ThreadLocal变量调用get()方法或者set()方法，就会对Thread类中的threadLocals进行初始化，并且以当前ThreadLocal变量为键值，以ThreadLocal要保存的副本变量为value，存到threadLocals。然后在当前线程里面，如果要使用副本变量，就可以通过get方法在threadLocals里面查找。

- 利用阻塞队列实现生产者消费者模式 http://www.cnblogs.com/tonyspark/p/3722013.html

- 讲解堆排序最好的文章 http://vickyqi.com/2015/08/14/%E6%8E%92%E5%BA%8F%E7%AE%97%E6%B3%95%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E5%A0%86%E6%8E%92%E5%BA%8F/

- 热补丁修复技术介绍 qq 空间的 https://mp.weixin.qq.com/s?__biz=MzI1MTA1MzM2Nw==&mid=400118620&idx=1&sn=b4fdd5055731290eef12ad0d17f39d4a
  注意问题就是在拆分 dex 的时候，有关联的两个类不在同一个 dex 中，存在一个 dex 校验问题，我们如何解决？解决这个校验问题，就要弄清楚校验过程出现在什么步骤。

- 非常全面的各大热修复技术的比较 https://zhuanlan.zhihu.com/p/25863920

- Binder 机制的讲解  http://www.jianshu.com/p/af2993526daf





​

​
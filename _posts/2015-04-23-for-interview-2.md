---
layout: post
title: 知识体系构建[Android]-系列二
date: 2015-04-23 19:53:26 +0800
categories: interview
---

个人知识体系构建第二个系列，主要是框架和源码的学习。

> 目录概览

- Https 安全
- Retrofit2 框架浅析
- OkHttp 框架浅析
- Glide 框架浅析
- ImageLoader 框架
- Butterknife 框架浅析
- JNI 开发过程
- 热修复的尝试
- 事件分发机制
- dex 文件格式，以及 JVM 能否加载 dex 文件
- JDK1.8 新增加的特性
- 枚举写出来的单利模式为什么安全
- ThreadLocal 底层实现
- hash 如何更加散列
- Java 线程之间通信方式
- Android 中 service 相关Service
- 操作系统中和数据结构中堆栈之间的区别
- Java8 函数式编程和 Lambda 表达式

> 具体内容

#### Https 为什么安全

https://zhuanlan.zhihu.com/p/25324735?utm_source=qq&utm_medium=social

- 对称加密

  对称加密是最基本的，但是很不安全。

- 公钥加密

  利用公钥进行加密，私钥进行解密。

- 消息摘要  

  消息摘要是用来判断数据完成性的一个算法，其加密过程是一个不可逆过程。常见的有 SHA、MD5.

- 消息验证

  消息认证码（message authentication code）是一种可以确认消息完整性并进行认证（消息认证是指确认消息来自正确的发送者）的技术，简称 MAC。消息认证码可以简单理解为一种与密钥相关的单向散列函数。

- 数据签名

- 公约证书

#### 热修复实践过程

http://www.jianshu.com/p/c36c9e0ca3fe

微信热补丁技术的演进过程：https://zhuanlan.zhihu.com/p/22239844

#### Okhttp

http://www.jianshu.com/p/a3334b931ae4#

![](http://ww1.sinaimg.cn/large/b10d1ea5ly1fewpvcloh5j20yg0e5t9o.jpg)

一般情况下，构造 Request 和 HttpClient 还是比较简单的，比较复杂的是发送请求的过程。在最后都讲请求交给了拦截器 Interceptor。okHTTP 通过一个 chain 将我们加入的拦截器串联在一起，每个拦截器只自己起作用。看一下拦截器的接口：

```java
public interface Interceptor {
  //只有一个接口方法
  Response intercept(Chain chain) throws IOException;
    //Chain大概是链条的意思
  interface Chain {
    // Chain其实包装了一个Request请求
    Request request();
    //获得Response
    Response proceed(Request request) throws IOException;
    //获得当前网络连接
    Connection connection();
  }
}
```

最后，总结一下这个请求动作发生在CallServerInterceptor（也就是最后一个Interceptor）中，而且其中还涉及到Okio这个io框架，通过Okio封装了流的读写操作，可以更加方便，快速的访问、存储和处理数据。最终请求调用到了socket这个层次,然后获得Response。

最近看到一个比较好的解释图 http://huachao1001.github.io/article.html?uOZGa337

![](http://ww1.sinaimg.cn/large/b10d1ea5ly1fexr9lw7mfj20rs18gdhq.jpg)

#### Retrofit2

https://zhuanlan.zhihu.com/p/21662195

从Retrofit的使用来看，Retrofit就是充当了一个适配器（Adapter）的角色：将一个Java接口*翻译*成一个Http请求，然后用OkHttp去发送这个请求。Retrofit非常巧妙的用注解来描述一个HTTP请求，将一个HTTP请求抽象成一个Java接口，然后用了Java动态代理的方式，动态的将这个接口的注解“翻译”成一个HTTP请求，最后再执行这个HTTP请求。执行这个 call 请求的其实本质上还是 okHttp。从 Retrofit 如何结合 Rxjava 角度来说一下 Retrofit 的好处，看下面这样的代码，就非常容易结合一些 Converter 了。

```java
Retrofit retrofit = new Retrofit.Builder()
  .baseUrl("https://api.github.com")
  .addConverterFactory(ProtoConverterFactory.create())
  .addConverterFactory(GsonConverterFactory.create())
  .addCallAdapterFactory(RxJavaCallAdapterFactory.create())
  .build();
```

#### Butterknife

http://brucezz.itscoder.com/use-apt-in-android

一般构建这种编译期注解的项目分为四个部分，分别是注解相关模块，注解处理器，API 接口模块，示例 demo。其本质上还是利用 APT 技术，根据注解利用 Javapoet 和 auto-service 来在程序编译期间生成代码，从而能够让主程序去调用相关模块。当然，为了能达到最好的效果，用一点反射也是可以的，个人感觉目前影响程序的效率，大部分还是我们的编码水平。

#### Android 事件分发机制

http://www.infoq.com/cn/articles/android-event-delivery-mechanism

当一个 view 需要处理事件时，若设置了 OnTouchListener ，则OnTouchLister 中的 OnTouch 方法会被调用，这个时候事件如何处理，要看 OnTouch 的返回值，如果返回 false ，则当前的 view 的 ontouchevent 方法会被调用；如果返回 true，那么 ontouchevent 则不会调用。由此可见给 view 设置的ontouchlisterner 其优先级要你 ontouchevent 高。在 ontouchevent 中，如果当前设置了 onclicklistener，那么它的 onclick 方法会被调用。可以看出，平常使用的 onclicklistener 其优先级最低，处于事件传递的最尾。

事件传递先传递到 viewgroup，再分发到 view。但是通过 requestDisallowInterceptTouchEvent 方法可以干预父元素的分发过程。view 的 ontouchevent 默认会消费事件，除非它是不可点击的。

![](http://ww1.sinaimg.cn/large/b10d1ea5ly1fexihki4n1j20ao05zwew.jpg)

#### Activity 的启动流程

http://huachao1001.github.io/article.html?feMdeAAE

#### Dex 文件的结构

DEX 文件这个截图出处我忘记了，找到的话会补上链接的。另外 dex 文件能否在 JVM 虚拟机上执行，或者 Class 文件能否在 Dalvik 虚拟机上执行？其实答案都是否定的，尽管 dex 和 class 文件有着很大关系，但是 Dalvik 并不是一个 Java 虚拟机，并没有遵循 Java 虚拟机的规范，使用的是寄存器架构，而不是 JVM 中常见的栈架构。

![](http://ww1.sinaimg.cn/large/b10d1ea5ly1fexrbk7z05j20m80mqmz1.jpg)

#### TCP 拥塞解决策略

http://wetest.qq.com/lab/view/246.html

#### 红黑树的结构和特性

http://www.cnblogs.com/skywang12345/p/3245399.html

红黑树始终是一个二叉搜索树，这种特性保证了其插入元素是有顺序的特性。

![](http://ww1.sinaimg.cn/large/b10d1ea5ly1fexs8kjn9ej20ps0ckjud.jpg)

* 每个节点或者是黑色，或者是红色。
* 根节点是黑色。
* 每个叶子节点（NIL）是黑色。 [注意：这里叶子节点，是指为空(NIL或NULL)的叶子节点！
* 如果一个节点是红色的，则它的子节点必须是黑色的。
* 从一个节点到该节点的子孙节点的所有路径上包含相同数目的黑节点。

#### Java 中 JDK1.7 与 1.8 的区别

- Java8 给接口添加了一个默认非抽象方法，用 dedault 实现，这一点即接口的默认实现，这样就减少了无谓的继承了。
- Lambda 来实现传统的内名内部类的表达方式
- 函数式接口

#### 枚举来实现单利模式的黑魔法

https://www.zhihu.com/people/pang-pang-37-37/answers

```java
public enum EasySingleton{
    INSTANCE;
}
```

枚举其实是 Java 的语法糖，反编译文件会发现枚举类型是个实实在在的类，并且编译器还加上了 static final 来修饰，并且编译器还偷偷的给枚举类的构造方法加上了 private 来修饰。

#### ThreadLocal 实现机制

http://www.jianshu.com/p/a31f6d889647

- Thread 保存线程变量的副本，研究的载体
- ThreadLocal 研究的主要对象
- ThreadLocalMap 与 Map 一点关系没有，内部实现机制跟 Map 很像
- ThreadLocalMap.Entry 它看成是保存键值对的对象，其本质上是一个WeakReference<ThreadLocal>对象

ThreadLocal 的 set 过程，如下源代码所示：

```java
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}
```

先拿到保存键值对的ThreadLocalMap对象实例map，如果map为空,则去创建一个ThreadLocalMap对象并赋值给map，并把键值对保存到map中。首先是拿到当前线程实例t，任何将t作为参数获取ThreadLocalMap对象。为什么需要通过Thread类来获取ThreadLocalMap对象呢？Thread类和ThreadLocalMap有什么联系？参见getMap(Thread t)`函数的实现。直接说就是通过 getMap 去获取当前线程的 threadlocal 属性，而该属性声明如下。这样大致就明白怎么回事了。

```java
ThreadLocal.ThreadLocalMap threadLocals = null;
```

最后，用一张图来表示这个关系，就能明白了。

![](http://ww1.sinaimg.cn/large/b10d1ea5ly1fevovj33e4j20ng0jbabn.jpg)

#### 回调 Demo

定义回调接口

```java
interface INotice {  
    public void tallBossOk(String strAsk);  
} 
```

员工类

```java
public class Staff implements INotice{    
    private Boss boss;  
    public Staff(Boss boss){  
        this.boss = boss;  
    }  
    public void startJob(){  
        try {  
            Thread.sleep(2000);  
        } catch (InterruptedException e) {  
            // TODO Auto-generated catch block  
            e.printStackTrace();  
        }  
        askResult("现在告诉老板我们完成工作了");  
    }  
    public void askResult(String str){  
        System.out.println(str);  
        boss.callBossMothod(Staff.this);//A调用B  
    }  
    public void tallBossOk(String strAsk) {  
        // TODO Auto-generated method stub  
        System.out.println("老板回应说"+strAsk);  
    }      
}  
```

老板类

```java
public class Boss {  
    public void callBossMothod(INotice iNotice){  
        iNotice.tallBossOk("行，你可以走了");//B调用A  
    }  
}  
```

测试函数

```java
public class Test {  
    public static void main(String[] args) {  
        // TODO Auto-generated method stub  
        Boss boss = new Boss();  
        Staff staff = new Staff(boss);  
        System.out.println("现在开始工作");  
        staff.startJob();  
    }  
} 
```

#### CMS收集器 和 G1 收集器

https://www.zhihu.com/people/pang-pang-37-37/answers?page=3

http://huachao1001.github.io/article.html?0wugT117

CMS(Concurrent Mark Sweep) 收集器基于 “标记-清除”算法实现的，主要运作过程分为以下四个步骤。CMS 并发收集、低停顿。

1. 初始标记
2. 并发标记
3. 重新标记
4. 并发清除

CMS 主要的缺点如下所示：

> - `CMS`收集器对`CPU`资源非常敏感。在并发阶段，它虽然不会导致用户线程停顿，但是会因为占用一部分线程（或者说`CPU`资源）而导致应用程序变慢，总吞吐量会降低。
> - `CMS`收集器无法处理浮动垃圾，可能出现`Concurrent Mode Failure`失败而导致另一次`Full GC`产生。由于`CMS`并发清理阶段用户线程还在运行着，伴随程序运行自然就还会有新的垃圾不断产生，这一部分垃圾出现在标记过程之后，`CMS`无法再当次收集中处理掉它们，治好留待下一次`GC`时再清理掉。这部分垃圾就称为“浮动垃圾”。因此，`CMS`不能像其他收集器那样等到老年代几乎完全被填满再进行手机，`CMS`需要预留一部分空间。
> - 由于CMS基于“标记-清除”算法，意味着收集结束时会有大量空间碎片产生。

G1（`Garbage First`）收集器是当今收集器技术发展的最前沿成果之一。`G1`是面向服务端应用的垃圾收集器，与其他`GC`收集器相比，`G1`具备如下特点：

> - **并行与并发**：充分利用多`CPU`、多核环境下的硬件优势，使用多个`CPU`（`CPU`或`CPU`核心）来缩短`Stop-The-World`停顿时间。部分其他收集器需要停顿`Java`线程执行的`GC`动作，`G1`仍然能通过并发方式让`Java`程序继续执行。
> - **分代收集**：与其他收集器一样，分代概念在G1中依然得以保存。
> - **空间整合**：与`CMS`的“标记-清理”算法不同，`G1`从整体上看是基于“标记-整理”算法实现的收集器，从局部上看是基于“复制”算法实现的，这两种算法意味着`G1`运作期间不会产生内存空间碎片。
> - **可预测的停顿**：这是`G1`相对于`CMS`的另一大优势，降低停顿时间是`G1`和`CMS`共同关注点，但`G1`除了追求低停顿外，还能建立可预测的停顿时间模型，能让使用者明确指定一个长度为`M`毫秒的时间片段内，消耗在垃圾收集上的时间不得从超出`N`毫秒，这几乎已经是实时`Java`的垃圾收集器的特征了。

`G1`运作大致可划分为以下几个步骤：

1. 初始标记
2. 并发标记
3. 最终标记
4. 筛选回收

#### NIO 和 BIO 的比较

http://qindongliang.iteye.com/blog/2018539

#### HashMap 如何更加散列

http://www.procedurego.com/article/137531.html

http://www.jianshu.com/p/8b372f3a195d  推荐这个 HashMap 必问到的。

jdk1.8 中的 hashCode 函数如下所示：

```java
public int hashCode() {
        int h = hash;
        if (h == 0 && value.length > 0) {
            char val[] = value;
            for (int i = 0; i < value.length; i++) {
                h = 31 * h + val[i];
            }
            hash = h;
        }
        return h;
    }
```

通过 hash 数值找到数组索引。其中h是hash值，length是数组的长度，这个按位与的算法其实就是h%length求余，一般什么情况下利用该算法，典型的分组。例如怎么将100个数分组16组中，就是这个意思。应用非常广泛。

```java
static int indexFor(int h, int length) { 
    return h & (length-1); 
}
```

利用一个实例来讲解以下上述索引寻址的缺点：

```java
int h=15,length=16;
        System.out.println(h & (length-1));
        h=15+16;
        System.out.println(h & (length-1));
        h=15+16+16;
        System.out.println(h & (length-1));
        h=15+16+16+16;
        System.out.println(h & (length-1));
```

输出的结构都是 15，为什么会这样呢，其实这里涉及到一个高低位的原理。换成二级制，就发现了，在做按位与操作的时候，后面的始终是低位在做计算，高位不参与计算，因为高位都是0。这样导致的结果就是只要是低位是一样的，高位无论是什么，最后结果是一样的，如果这样依赖，hash碰撞始终在一个数组上，导致这个数组开始的链表无限长，那么在查询的时候就速度很慢，又怎么算得上高性能的啊。所以hashmap必须解决这样的问题，尽量让key尽可能均匀的分配到数组上去。避免造成Hash堆积。以下为 JDK1.8 中的二次 hash 过程，也就是扰动函数来解决我们这个存在的 hash 堆积的问题。关于这个位移 16的操作，非常推荐你看 [知乎讨论](https://www.zhihu.com/question/20733617)

```java
static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
```

上述这个代码，分解一下看就是下面这样，本质上个人感觉还是利用异或操作，相同为 0 ，不同为 1.

http://www.procedurego.com/article/137531.html

```java
static final int hash(Object key) {
    if(key == null)
        return 0;
        int h = key.hashCode();
        int temp = h >>> 16;
        int newHash = h ^ temp;
        return newHash;
    }
```



解决 hash 冲突的四种方式

- 开放定址法 线性探测再散列，二次探测再散列，伪随机探测再散列）
- 再哈希法
- 链地址法
- 建立一 公共溢出区

#### Java 到底值传递还是引用传递

http://6924918.blog.51cto.com/6914918/1283761

- 基本类型时候，传递的是值的拷贝
- 而 String 类很奇怪，一般也是作为基本类型来传递
- 对象作为参数传递时，是把对象在内存中的地址拷贝了一份传给了参数

#### 接口与抽象类之间异同

- 都不能被实例化
- 都可以包含抽象方法，但是实现/继承的子类必须实现抽象方法
- 接口只能定义静态常量，但是抽象类普通成员变量和静态常量都可以
- 抽象类可以包含普通方法，但是接口不可以

#### 创建线程三种方式对比

继承 Thread或者实现Runnable、Callable 接口都可以实现多线程，实现 Callable或者Runnable接口的方式基本相同，只是 Callable 接口定义的方法有返回值，可以声明抛出异常。

- 采用实现 Runnable、Callable接口方式创建多线程的优缺点
  - 实现了接口，但是还是可以继承其他类
  - 多个线程共享同一个 target 类
  - 缺点就是编程稍微复杂，访问当前线程有点麻烦
- 采用继承 Thread 类
  - 不支持多继承
  - 编写简单，获取当前线程直接使用 this 即可

#### Java 线程间通信

* 传统的线程通信方式

可以借助于 Object 类的 wait 、notify、notifyall 三个方法。但是这三个方法必须由同步监视器synchronized对象来调用。

1. wait 导致当前线程等待，直到其他线程调用该同步监视器的 notify/notifyall 方法来唤醒该线程
2. notify 唤醒在此同步监视器上等待的单个线程，若存在多个，任意选择其中一个
3. notifyAll 唤醒所有的在此同步监视器上等待的线程

* 使用 condition 来控制线程间通信

如果不使用 sync 关键字来进行同步，而使用 lock ，则系统中不存在隐式不同监视器，则只能调用 Java 提供的 condition 类来保持协调

1. await 类似于 wait
2. singal 类似于 notify
3. singalAll 类似于 notifyAll

使用阻塞队列来进行线程间通信

Java5 提供的 BQ 接口，队列满/空，则生产者/消费者，往其中添加/取出都会存在阻塞。也就是说利用这个 BQ 可以很方便的实现生产者消费者模式。

1. ArrayBQ 基于数组实现的 BQ 队列
2. LInkedBQ 基于链表实现的 BQ 队列
3. PriorityBQ 具备优先级的队列，而并不是传统的取队尾的元素。

#### Android 中 Service

http://huachao1001.github.io/article.html?tlRlR66O

![](http://ww1.sinaimg.cn/large/b10d1ea5ly1fewyy7ffdnj20at0e3abt.jpg)

``` java
public class ExampleService extends Service { 
    int mStartMode;       // 标识服务被杀死后的处理方式 
    IBinder mBinder;      // 用于客户端绑定的接口 
    boolean mAllowRebind; // 标识是否使用onRebind 
 
    @Override 
    public void onCreate() { 
        // 服务正被创建 
    } 
    @Override 
    public int onStartCommand(Intent intent, int flags, int startId) { 
        // 服务正在启动，由startService()调用引发 
        return mStartMode; 
    } 
    @Override 
    public IBinder onBind(Intent intent) { 
        // 客户端用bindService()绑定服务 
        return mBinder; 
    } 
    @Override 
    public boolean onUnbind(Intent intent) { 
        // 所有的客户端都用unbindService()解除了绑定 
        return mAllowRebind; 
    } 
    @Override 
    public void onRebind(Intent intent) { 
        // 某客户端正用bindService()绑定到服务, 
        // 而onUnbind()已经被调用过了 
    } 
    @Override 
    public void onDestroy() { 
        // 服务用不上了，将被销毁 
    } 
}
```

onStartCommand() 方法必须返回一个整数，这个整数描述系统在杀死服务之后该如何继续执行，其返回值必须是下面这是三个常量之一：

> - START_NOT_STICKY：如果系统在onStartCommand()返回后杀死了服务，则不会重建服务了，除非还存在未发送的intent。 当服务不再是必需的，并且应用程序能够简单地重启那些未完成的工作时，这是避免服务运行的最安全的选项.
> - START_STICKY：如果系统在onStartCommand()返回后杀死了服务，则将重建服务并调用onStartCommand()，但不会再次送入上一个intent， 而是用null intent来调用onStartCommand() 。除非还有启动服务的intent未发送完，那么这些剩下的intent会继续发送。 这适用于媒体播放器（或类似服务），它们不执行命令，但需要一直运行并随时待命。　
> - START_REDELIVER_INTENT：如果系统在onStartCommand()返回后杀死了服务，则将重建服务并用上一个已送过的intent调用onStartCommand()。任何未发送完的intent也都会依次送入。这适用于那些需要立即恢复工作的活跃服务，比如下载文件。

#### Activity 的启动过程

http://huachao1001.github.io/article.html?feMdeAAE

#### 操作系统和数据结构中堆栈的区别

http://blog.csdn.net/speedme/article/details/22943191

- **栈（操作系统）**：由编译器自动分配释放 ，存放函数的参数值，局部变量的值等。其操作方式类似于数据结构中的栈，栈使用的是一级缓存， 他们通常都是被调用时处于存储空间中，调用完毕立即释放
- **堆（操作系统）**： 一般由程序员分配释放， 若程序员不释放，程序结束时可能由OS回收，分配方式倒是类似于链表。堆则是存放在二级缓存中，生命周期由虚拟机的垃圾回收算法来决定（并不是一旦成为孤儿对象就能被回收）。所以调用这些对象的速度要相对来得低一些
- **堆（数据结构）**：堆可以被看成是一棵树，如：堆排序
- **栈（数据结构）**：一种后进先出的数据结构

#### Java8 中函数式编程

http://blog.oneapm.com/apm-tech/226.html

为什么会出现函数式编程和 lambda 表达式，主要还是因为在 Java 中传递函数参数的不方便。常见的 lambda 表达式如下所示：

``` java
(int a, int b) -> {  return a + b; }

() -> System.out.println("Hello World");

(String s) -> { System.out.println(s); }

() -> 42

() -> { return 3.1415 };
```

在函数式编程语言中，函数是一等公民。这种编程语言，提供了一种强大的功能，就是闭包。闭包的好处要理解一下，而在 Java 语言中最接近闭包的就是 lambda 表达式了。另外更加引申就涉及到了函数式接口了，这一块概念要与 lambda 区分开来。
---
layout: post
title: 深入理解 Android 之 Picasso 源码解析
date: 2017-08-29 21:20:24 +0800
categories: 
---

分析一个图片加载库的原理，选择代码量不多且比较容易分析的Picasso来进行分析。

### 基本使用

下面这行代码就是它最基本的使用啦：

``` java
Picasso.with(context).load(“url”).into(imageView);
```
当然也可以对它进行基本的图片转换操作了

```java
Picasso.with(context)
  .load(url)
  // 裁剪图片尺寸
  .resize(100, 100)
  // 设置图片圆角
  .centerCrop()
  .into(imageView)
```

占位图替代和加载失败之后所作出的操作

```java
Picasso.with(context)
    .load(url)
    // 加载过程中的占位图
    .placeholder(R.drawable.user_placeholder)
    // 如果重试3次还是无法成功加载图片，则用错误占位符图片显示。
    .error(R.drawable.user_placeholder_error)
    .into(imageView);
```

从不同地方加载图片

``` java
// 加载资源文件
Picasso.with(context).load(R.drawable.landing_screen).into(imageView1);
// 加载本地文件
Picasso.with(context).load(new File("/images/oprah_bees.gif")).into(imageView2);
```

给图片设置tag

``` java
// 加载图片并设置tag,可以通过tag来暂定或者继续加载,可以用于当ListView滚动是暂定加载.停止滚动恢复加载.
Picasso.with(this).load("url").tag(mContext).into(imageView);
Picasso.with(this).pauseTag(mContext);
Picasso.with(this).resumeTag(mContxt);
```

更多的特性，如Debug模式可以参见其[官网](http://square.github.io/picasso/)介绍啦，其实也就是API的使用，了解一个框架原理之前，最好能熟知它的用法。

### 特性优势


* 能够根据不同网络情况，修改线程池ExecutorService的线程个数，4g、3g、wifi情况下表现不同
* 创建默认的监控器，统计下载时长、缓存命中率等
* 通过BitmapHunter这个核心类中的run方法来下载图片，并将图片解码成Bitmap，然后做一些转换操作，剪裁啊，旋转啊等 
* 显示图片加载来源，设置不同的tag
* 轻量级、易扩展

### 框架概览

* 请求分发模块。负责封装请求,对请求进行优先级排序,并按照类型进行分发。

* 缓存模块。通常包括一个二级的缓存，内存缓存、磁盘缓存。并预置多种缓存策略。

* 下载模块。负责下载网络图片。

* 监控模块。负责监控缓存命中率、内存占用、加载图片平均耗时等。

* 图片处理模块。负责对图片进行压缩、变换等处理。

* 本地资源加载模块。负责加载本地资源，如assert、drawable、sdcard等。

* 显示模块。负责将图片输出显示。

### 步骤框架

下图来自网图codekk源码分析系列，特此声明：

![](http://ww1.sinaimg.cn/large/006dXScfly1fj3qbsq0xxj30hs0edab3.jpg)

​	**整个执行步骤**：

创建->入队->执行->解码->变换->批处理->完成->分发->显示

### 深入源码

接下来就重点分析Picasso的源码了，分析源码得找好切入点，细节太多了。本文就按照Picasso的用法即with-load-into这个流程来了，主要还是解析怎么从一个uri或者res到能够将Bitmap显示在ImageView上的一个过程。

#### Picasso#with

这个with方法是入口，没什么好讲的，返回一个单利Builder而已，重要的是Builder.builder有哪些操作：

```java
  public static Picasso with() {
    if (singleton == null) {
      synchronized (Picasso.class) {
        if (singleton == null) {
          if (PicassoProvider.context == null) {
            throw new IllegalStateException("context == null");
          }
          singleton = new Builder(PicassoProvider.context).build();
        }
      }
    }
    return singleton;
  }
```

#### Picasso#Builder#build

``` java
public Picasso build() {
      Context context = this.context;

      if (downloader == null) {
       // 默认就是 okhttp下载器
        downloader = new OkHttp3Downloader(context);
      }
      if (cache == null) {
        // 配置内存缓存，大小为手机内存的15%
        cache = new LruCache(context);
      }
      if (service == null) {
        // 配置Picaso 线程池，核心池大小为3
        service = new PicassoExecutorService();
      }
      if (transformer == null) {
        // 配置请求转换器，默认的请求转换器没有做任何事，直接返回原请求
        transformer = RequestTransformer.IDENTITY;
      }

      // 创建一个统计类
      Stats stats = new Stats(cache);
      // 分发器 ，初始化调度者
      Dispatcher dispatcher = new Dispatcher(context, service, HANDLER, downloader, cache, stats);

      // 返回一个Picasso实例
      return new Picasso(context, dispatcher, cache, listener, transformer, requestHandlers, stats,
          defaultBitmapConfig, indicatorsEnabled, loggingEnabled);
    }
```
大部分初始化变量都已经在上述代码中解释过了，注意下**requestHandlers**其作用是创建处理器集合，用于处理不同的**加载请求**，其初始化过程是在Picasso的构造器中。

``` java
// ResourceRequestHandler needs to be the first in the list to avoid
// forcing other RequestHandlers to perform null checks on request.uri
// to cover the (request.resourceId != 0) case.
allRequestHandlers.add(new ResourceRequestHandler(context));
    if (extraRequestHandlers != null) {
      allRequestHandlers.addAll(extraRequestHandlers);
    }
// 从contactsPhoto加载图片
allRequestHandlers.add(new ContactsPhotoRequestHandler(context));
// 从MediaStore加载
allRequestHandlers.add(new MediaStoreRequestHandler(context));
allRequestHandlers.add(new ContentStreamRequestHandler(context));
// 本地Asset加载
allRequestHandlers.add(new AssetRequestHandler(context));
allRequestHandlers.add(new FileRequestHandler(context));
// 从网络加载图片
allRequestHandlers.add(new NetworkRequestHandler(dispatcher.downloader, stats));
requestHandlers = Collections.unmodifiableList(allRequestHandlers);
```

接下来按照Picasso的用法就应该去看load方法了：

#### Picasso#load

``` java
  public RequestCreator load(@Nullable Uri uri) {
    return new RequestCreator(this, uri, 0);
  }

  public RequestCreator load(@Nullable String path) {
    if (path == null) {
      return new RequestCreator(this, null, 0);
    }
    if (path.trim().length() == 0) {
      throw new IllegalArgumentException("Path must not be empty.");
    }
    return load(Uri.parse(path));
  }

  public RequestCreator load(@NonNull File file) {
    if (file == null) {
      return new RequestCreator(this, null, 0);
    }
    return load(Uri.fromFile(file));
  }

  public RequestCreator load(@DrawableRes int resourceId) {
    if (resourceId == 0) {
      throw new IllegalArgumentException("Resource ID must not be zero.");
    }
    return new RequestCreator(this, null, resourceId);
  }
```
从上述知道load这个方法有多种重载，其作用是为了从不同地方加载资源。核心还是**RequestCreator**这个类。接下来的分析自然要到 RequestCreator中。并且load只是把uri或者path传递过里啊，后续就没啥作用了，下一步继续要分析的就是into方法了。

#### RequestCreator#into

同样，into方法也有多个重载版本，选择target为Imageview作为分析对象

``` java
  public void into(ImageView target, Callback callback) {
    long started = System.nanoTime();
    // 检查是否为主线程，这里为什么需要去检测是否是主线程？
    checkMain();

    // Image存放的target不能为null
    if (target == null) {
      throw new IllegalArgumentException("Target must not be null.");
    }
	
    // 不存在
    if (!data.hasImage()) {
      picasso.cancelRequest(target);
      if (setPlaceholder) {
        setPlaceholder(target, getPlaceholderDrawable());
      }
      return;
    }

    // 是否调用了fit(),如果是，代表需要将image调整为ImageView的大小
    if (deferred) {
      // fit不能有resize一起使用
      if (data.hasSize()) {
        throw new IllegalStateException("Fit cannot be used with resize.");
      }
      int width = target.getWidth();
      int height = target.getHeight();
      if (width == 0 || height == 0 || target.isLayoutRequested()) {
        if (setPlaceholder) {
          setPlaceholder(target, getPlaceholderDrawable());
        }
        picasso.defer(target, new DeferredRequestCreator(this, target, callback));
        return;
      }
      // resize模式
      data.resize(width, height);
    }

    // 创建Request
    Request request = createRequest(started);
    String requestKey = createKey(request);

    // 根据缓存策略从缓存中读取
    if (shouldReadFromMemoryCache(memoryPolicy)) {
      Bitmap bitmap = picasso.quickMemoryCacheCheck(requestKey);
      if (bitmap != null) {
        picasso.cancelRequest(target);
        setBitmap(target, picasso.context, bitmap, MEMORY, noFade, picasso.indicatorsEnabled);
        if (picasso.loggingEnabled) {
          log(OWNER_MAIN, VERB_COMPLETED, request.plainId(), "from " + MEMORY);
        }
        if (callback != null) {
          callback.onSuccess();
        }
        return;
      }
    }

    // 设置Placeholder
    if (setPlaceholder) {
      setPlaceholder(target, getPlaceholderDrawable());
    }

    // 构建一个Action对象,由于我们是往ImageView里加载图片,所以这里创建的是一个ImageViewAction对象.
    Action action =
        new ImageViewAction(picasso, target, request, memoryPolicy, networkPolicy, errorResId,
            errorDrawable, requestKey, tag, callback, noFade);

    // action提交到
    picasso.enqueueAndSubmit(action);
  }
```
讲道理此时，应该从**picasso.enqueueAndSubmit(action);**跟进去，但是需要先去了解Action代表什么，跟之前的Request有关系吗？

* Request关注的是请求本身，比如请求的源、id、开始时间、图片变换配置、优先级等等，而Action则代表的是一个加载任务，所以不仅需要 
  Request对象的引用，还需要Picasso实例，是否重试加载等等

``` java
public final class Request {
  private static final long TOO_LONG_LOG = TimeUnit.SECONDS.toNanos(5);
  // 给request赋予的唯一的ID
  int id;
  // request提交到线程池中的时间记录
  long started;
  // 该request的网络请求策略
  int networkPolicy;
  // 加载源头
  public final Uri uri;  
  public final int resourceId;
  public final String stableKey;
  /** List of custom transformations to be applied after the built-in transformations. */
  public final List<Transformation> transformations;
  /** Target image width for resizing. */
  public final int targetWidth;
  /** Target image height for resizing. */
  public final int targetHeight;
  public final boolean centerCrop;
  public final int centerCropGravity;
  public final boolean centerInside;
  public final boolean onlyScaleDown;
  /** Amount to rotate the image in degrees. */
  public final float rotationDegrees;
  /** Rotation pivot on the X axis. */
  public final float rotationPivotX;
  /** Rotation pivot on the Y axis. */
  public final float rotationPivotY;
  /** Whether or not {@link #rotationPivotX} and {@link #rotationPivotY} are set. */
  public final boolean hasRotationPivot;
  /** True if image should be decoded with inPurgeable and inInputShareable. */
  public final boolean purgeable;
  /** Target image config for decoding. */
  public final Bitmap.Config config;
  // request的请求优先级
  public final Priority priority;
```

* Action中WeakReference<T> target,它持有的是Target(ImageView..)的弱引用，这样可以保证加载时间很长的情况下 也不会影响到Target的回收了.这些Action类的实现类不仅保存了这次加载需要的所有信息,还提供了加载完成后的回调方法.也是由子类实现并用来完成不同的调用的

``` java
  abstract class Action<T> {
  // picasso单例
  final Picasso picasso;
  // 当前任务的request  
  final Request request;  
  // 目标，imageview
  final WeakReference<T> target; 
  // 是否渐进渐出 
  final boolean noFade;  
  // 内存缓存策略
  final int memoryPolicy;  
  // 网络缓存策略
  final int networkPolicy;  
  // 错误占位图资源id
  final int errorResId;  
  // 错误占位图资源drawable
  final Drawable errorDrawable; 
  // 内存缓存的key 
  final String key; 
  // dispatcher的任务标识
  final Object tag; 
  // 是否再次放到任务队列中
  boolean willReplay; 
  // 是否取消 
  boolean cancelled;  
  // Bitmap加载成功回调
  abstract void complete(Bitmap result, Picasso.LoadedFrom from);
  // 加载失败回调
  abstract void error();
```

接下来继续前面的步骤，看看如何submit一个Action

``` java
  void enqueueAndSubmit(Action action) {
    // 从action中拿到target
    Object target = action.getTarget();
    if (target != null && targetToAction.get(target) != action) {
      // This will also check we are on the main thread.
      cancelExistingRequest(target);
      targetToAction.put(target, action);
    }
    submit(action);
  }

  void submit(Action action) {
    dispatcher.dispatchSubmit(action);
  }
```

#### Dispatcher#performSubmit

紧接着上面的代码我们需要看一下Dispatcher这个类相关的信息，如其构造函数：

``` java
Dispatcher(Context context, ExecutorService service, Handler mainThreadHandler,
      Downloader downloader, Cache cache, Stats stats)
```

可以看出声明一个dispatcher需要以下元素：

* 线程池ExecutorService，默认为3，但是会随着网络的不同而变化
* Handler处理器
* 下载器Downloader
* Cache
* Stats监控器

还需要关注一下内部类,是一个继承handlerThrea的线程：

``` java
 static class DispatcherThread extends HandlerThread {
    DispatcherThread() {
      super(Utils.THREAD_PREFIX + DISPATCHER_THREAD_NAME, THREAD_PRIORITY_BACKGROUND);
    }
  }
```
那这个内部类的作用是什么呢，注意到了Dispatcher中的构造器中：

``` java
 this.handler = new DispatcherHandler(dispatcherThread.getLooper(), this);
```
原来如此，这样后续的handler操作都是在子线程中进行了，避免阻塞主线程的消息队列了。紧接着还是从 ` dispatcher.dispatchSubmit(action);`继续深入：

``` java
 void dispatchSubmit(Action action) {
    handler.sendMessage(handler.obtainMessage(REQUEST_SUBMIT, action));
  }
```

发送到handler，根据消息机制的Case之后会调用下面这儿的performSubmit方法

``` java
void performSubmit(Action action, boolean dismissFailed) {
    if (pausedTags.contains(action.getTag())) {
      pausedActions.put(action.getTarget(), action);
      if (action.getPicasso().loggingEnabled) {
        log(OWNER_DISPATCHER, VERB_PAUSED, action.request.logId(),
            "because tag '" + action.getTag() + "' is paused");
      }
      return;
    }

    // 获取 BitmapHunter对象，从缓存中
    BitmapHunter hunter = hunterMap.get(action.getKey());
    if (hunter != null) {
      hunter.attach(action);
      return;
    }

    // 判断线程池有咩有关闭
    if (service.isShutdown()) {
      if (action.getPicasso().loggingEnabled) {
        log(OWNER_DISPATCHER, VERB_IGNORED, action.request.logId(), "because shut down");
      }
      return;
    }

    // 创建BitmapHunter对象
    hunter = forRequest(action.getPicasso(), this, cache, stats, action);
    hunter.future = service.submit(hunter);
    hunterMap.put(action.getKey(), hunter);
    if (dismissFailed) {
      failedActions.remove(action.getTarget());
    }

    if (action.getPicasso().loggingEnabled) {
      log(OWNER_DISPATCHER, VERB_ENQUEUED, action.request.logId());
    }
  }
```
上面代码中BitmapHunnter是一个Runnable对象，创建好之后会先判断线程池有没有关闭，没有关闭就丢进去让线程池执行，线程池接受到之后就会执行Runnable对象中的run方法，跟进去看一下：

#### BitmapHunter#run

```java
  @Override public void run() {
    try {
      // 更新当前线程名字
      updateThreadName(data);

      if (picasso.loggingEnabled) {
        log(OWNER_HUNTER, VERB_EXECUTING, getLogIdsForHunter(this));
      }

      // 调用hunt方法
      result = hunt();

      // 通过dispatcher处理结果
      if (result == null) {
        dispatcher.dispatchFailed(this);
      } else {
        dispatcher.dispatchComplete(this);
      }
    } catch (NetworkRequestHandler.ResponseException e) {
      
    } finally {
      Thread.currentThread().setName(Utils.THREAD_IDLE_NAME);
    }
  }
```
先进入到hunt方法中，看看返回是什么result？

#### BitmapHunter#hunt

``` java
 Bitmap hunt() throws IOException {
    Bitmap bitmap = null;
    // 先从缓存中获取
    if (shouldReadFromMemoryCache(memoryPolicy)) {
      bitmap = cache.get(key);
      if (bitmap != null) {
        // 统计命中率
        stats.dispatchCacheHit();
        loadedFrom = MEMORY;
        if (picasso.loggingEnabled) {
          log(OWNER_HUNTER, VERB_DECODED, data.logId(), "from cache");
        }
        return bitmap;
      }
    }

    // 缓存没有的话，再调用requestHandler.load，选择什么requesthandler就看具体情况了
    networkPolicy = retryCount == 0 ? NetworkPolicy.OFFLINE.index : networkPolicy;
    RequestHandler.Result result = requestHandler.load(data, networkPolicy);
    if (result != null) {
      loadedFrom = result.getLoadedFrom();
      exifOrientation = result.getExifOrientation();
      bitmap = result.getBitmap();

      // If there was no Bitmap then we need to decode it from the stream.
      if (bitmap == null) {
        Source source = result.getSource();
        try {
          // 压缩
          bitmap = decodeStream(source, data);
        } finally {
          try {
            //noinspection ConstantConditions If bitmap is null then source is guranteed non-null.
            source.close();
          } catch (IOException ignored) {
          }
        }
      }
    }

    if (bitmap != null) {
      if (picasso.loggingEnabled) {
        log(OWNER_HUNTER, VERB_DECODED, data.logId());
      }
      stats.dispatchBitmapDecoded(bitmap);
     // 变换操作
      if (data.needsTransformation() || exifOrientation != 0) {
        synchronized (DECODE_LOCK) {
          if (data.needsMatrixTransform() || exifOrientation != 0) {
            bitmap = transformResult(data, bitmap, exifOrientation);
            if (picasso.loggingEnabled) {
              log(OWNER_HUNTER, VERB_TRANSFORMED, data.logId());
            }
          }
          if (data.hasCustomTransformations()) {
            bitmap = applyCustomTransformations(data.transformations, bitmap);
            if (picasso.loggingEnabled) {
              log(OWNER_HUNTER, VERB_TRANSFORMED, data.logId(), "from custom transformations");
            }
          }
        }
        if (bitmap != null) {
          stats.dispatchBitmapTransformed(bitmap);
        }
      }
    }
    // 注意这里返回的bitmap
    return bitmap;
  }
```

到这里注意逻辑别乱，hunt方法是在run方法中间调用的，最后需要知道hunt方法返回结果之后，图片成功显示与否怎么办呢？在run方法中：

``` java
 // 通过dispatcher处理结果
if (result == null) {
   dispatcher.dispatchFailed(this);
  } else {
   dispatcher.dispatchComplete(this);
}
```

也是通过handler机制发送消息来更新的，最后辗转会到下面的performComplete这方法中

``` java
void performComplete(BitmapHunter hunter) {
    // 写入缓存，根据之前的缓存策略
    if (shouldWriteToMemoryCache(hunter.getMemoryPolicy())) {
      cache.set(hunter.getKey(), hunter.getResult());
    }
    // 从hunterMap中移除
    hunterMap.remove(hunter.getKey());
    // 批量处理hunter，等待200ms
    batch(hunter);
    if (hunter.getPicasso().loggingEnabled) {
      log(OWNER_DISPATCHER, VERB_BATCHED, getLogIdsForHunter(hunter), "for completion");
    }
  }
```
进入到上面的batch这个操作中查看一下，顾名思义是批处理的意思，看到下面的代码，其实操作还是放进batch#add这个函数中，本意是为了将操作缓存一下，等空闲的时候再拿出来处理，这样可以更有逻辑性，避免出现ANR现象

``` java
private void batch(BitmapHunter hunter) {
    if (hunter.isCancelled()) {
      return;
    }
    if (hunter.result != null) {
      hunter.result.prepareToDraw();
    }
    batch.add(hunter);
    if (!handler.hasMessages(HUNTER_DELAY_NEXT_BATCH)) {
      handler.sendEmptyMessageDelayed(HUNTER_DELAY_NEXT_BATCH, BATCH_DELAY);
    }
  }
```
发送HUNTER_DELAY_NEXT_BATCH消息,这个消息最后会出发performBatchComplete()方法,performBatchComplete()里则是通过mainThreadHandler将BitmapHunter的List发送到主线程处理,所以去看看mainThreadHandler的handleMessage()方法:

``` java
@Override public void handleMessage(final Message msg) {
      switch (msg.what) {
        ...
        case HUNTER_DELAY_NEXT_BATCH: {
          dispatcher.performBatchComplete();
          break;
        }
        ...
      }
}
```

顺其自然接着看:

#### Dispathcer#performBatchComplete

``` java
void performBatchComplete() {
    List<BitmapHunter> copy = new ArrayList<>(batch);
    // 清除batch
    batch.clear();
    mainThreadHandler.sendMessage(mainThreadHandler.obtainMessage(HUNTER_BATCH_COMPLETE, copy));
    logBatch(copy);
  }
```
这个mainThreadHandler是在Dispatcher实例化时由外部传递进来的，在前面的分析中看到，Picasso在通过Builder创建时会对Dispatcher进行实例化，在那个地方将主线程的handler传了进来，回到Picasso这个类，看到其有一个静态成员变量HANDLER，这样我们也就清楚了。

``` java
  static final Handler HANDLER = new Handler(Looper.getMainLooper()) {
    @Override public void handleMessage(Message msg) {
      switch (msg.what) {
        case HUNTER_BATCH_COMPLETE: {
          @SuppressWarnings("unchecked") List<BitmapHunter> batch = (List<BitmapHunter>) msg.obj;
          //noinspection ForLoopReplaceableByForEach
          for (int i = 0, n = batch.size(); i < n; i++) {
            // 对每个BitmapHunter采用complete方法
            BitmapHunter hunter = batch.get(i);
            hunter.picasso.complete(hunter);
          }
          break;
        }
        case REQUEST_GCED: {
          Action action = (Action) msg.obj;
          if (action.getPicasso().loggingEnabled) {
            log(OWNER_MAIN, VERB_CANCELED, action.request.logId(), "target got garbage collected");
          }
          action.picasso.cancelExistingRequest(action.getTarget());
          break;
        }
        case REQUEST_BATCH_RESUME:
          @SuppressWarnings("unchecked") List<Action> batch = (List<Action>) msg.obj;
          //noinspection ForLoopReplaceableByForEach
          for (int i = 0, n = batch.size(); i < n; i++) {
            Action action = batch.get(i);
            action.picasso.resumeAction(action);
          }
          break;
        default:
          throw new AssertionError("Unknown handler message received: " + msg.what);
      }
    }
  };
```
代码很长，重点关注`case HUNTER_BATCH_COMPLETE: `其对batch中存储的操作分步骤处理，进而进入到complete中，隐约感觉图片快要显示出来了,紧跟complete这个方法

``` java
void complete(BitmapHunter hunter) {
    Action single = hunter.getAction();
    List<Action> joined = hunter.getActions();

    boolean hasMultiple = joined != null && !joined.isEmpty();
    boolean shouldDeliver = single != null || hasMultiple;

    if (!shouldDeliver) {
      return;
    }

    Uri uri = hunter.getData().uri;
    Exception exception = hunter.getException();
    Bitmap result = hunter.getResult();
    LoadedFrom from = hunter.getLoadedFrom();

    // 分发单独的Action
    if (single != null) {
      deliverAction(result, from, single, exception);
    }

    // 有重复的
    if (hasMultiple) {
      //noinspection ForLoopReplaceableByForEach
      for (int i = 0, n = joined.size(); i < n; i++) {
        Action join = joined.get(i);
        // 注意在这里派发Action
        deliverAction(result, from, join, exception);
      }
    }

    if (listener != null && exception != null) {
      listener.onImageLoadFailed(this, uri, exception);
    }
  }
```
紧跟deliverAction方法中去看一看：

``` java
 private void deliverAction(Bitmap result, LoadedFrom from, Action action, Exception e) {
    if (action.isCancelled()) {
      return;
    }
    if (!action.willReplay()) {
      targetToAction.remove(action.getTarget());
    }
    if (result != null) {
      if (from == null) {
        throw new AssertionError("LoadedFrom cannot be null.");
      }
      // 回调action中的complete
      action.complete(result, from);
      if (loggingEnabled) {
        log(OWNER_MAIN, VERB_COMPLETED, action.request.logId(), "from " + from);
      }
    } else {
      // 回调action中的complete
      action.error(e);
      if (loggingEnabled) {
        log(OWNER_MAIN, VERB_ERRORED, action.request.logId(), e.getMessage());
      }
    }
  }
```

#### Action#complete

``` java
@Override public void complete(Bitmap result, Picasso.LoadedFrom from) {
    if (result == null) {
      throw new AssertionError(
          String.format("Attempted to complete action with no result!\n%s", this));
    }

    // imageview作为target，来提供图片显示的位置
    ImageView target = this.target.get();
    // 判断target是否为null
    if (target == null) {
      return;
    }

    Context context = picasso.context;
    boolean indicatorsEnabled = picasso.indicatorsEnabled;

    // setbitmap操作来将图片显示上去
    PicassoDrawable.setBitmap(target, context, result, from, noFade, indicatorsEnabled);

    // 回调成功接口callback
    if (callback != null) {
      callback.onSuccess();
    }
}
```

### 关注的点

本篇文章重点记录Picasso请求显示图片的过程，完整的分析做的还不是很全面。继续深入可以从下面的方法来挖掘这个库，篇幅有限，本篇就先到这里了。

* 缓存策略
* 预加载
* 图片变换








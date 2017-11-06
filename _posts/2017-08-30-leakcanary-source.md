---
layout: post
title: 深入理解 Android 之 LeakCanary 源码解析
date: 2017-08-30 08:41:17 +0800
categories: 
---

Android中的内存泄漏问题是性能优化的大头，不少开发者也为如何有效的检测出内存泄漏煞费苦心。今天来分析一下LeakCanary的原理，它的基本的使用看其官网介绍即可，非常简单、易上手。同时在了解了本篇文章之后，熟悉了Leakcanary的基本工作原理以及代码结构，强烈推荐去了解一些Blockcanary的用处以及工作原理。对理解整个Android系统的消息机制工作过程，非常有好处。

### 原理概览

LeakCanary的原理非常简单。正常情况下一个Activity在执行Destroy之后就要销毁，LeakCanary做的就是在一个Activity Destroy之后将它放在一个WeakReference中，然后将这个WeakReference关联到一个ReferenceQueue，查看ReferenceQueue是否存在Activity的引用，如果不在这个队列中，执行一些GC清洗操作，再次查看。如果不存在则证明该Activity泄漏了，之后Dump出heap信息，并用haha这个开源库去分析泄漏路径。


### 源码结构

1. leakcanary-watcher: 这是一个通用的内存检测器，对外提供一个 RefWatcher#watch(Object watchedReference),它不仅能够检测Activity，还能监测任意常规的 Java Object 的泄漏情况。

2. leakcanary-android: 这个 module 是与 Android 的接入点，用来专门监测 Activity 的泄漏情况，内部使用了 application#registerActivityLifecycleCallbacks 方法来监听 onDestory 事件，然后利用 leakcanary-watcher 来进行弱引用＋手动 GC 机制进行监控。

3. leakcanary-analyzer: 这个 module 提供了 HeapAnalyzer，用来对 dump 出来的内存进行分析并返回内存分析结果AnalysisResult，内部包含了泄漏发生的路径等信息供开发者寻找定位。

4. leakcanary-android-no-op: 这个 module 是专门给 release 的版本用的，内部只提供了两个完全空白的类 LeakCanary 和 RefWatcher，这两个类不会做任何内存泄漏相关的分析。因为 LeakCanary 本身会由于不断 gc 影响到 app 本身的运行，而且主要用于开发阶段的内存泄漏检测。因此对于 release 则可以 disable 所有泄漏分析。

### 流程分析

leakcanary的一个基本使用过程类似于下面的这个流程图：

![](http://ww1.sinaimg.cn/large/006dXScfly1fj22w7flt4j30z00mrtc0.jpg)

### 深入源码

按照常见的思路，从应用开始分析，开始跟进代码：

#### 01.LeakCanary#Install

``` java
 public static RefWatcher install(Application application,String processType, String remotePath,
      Class<? extends AbstractAnalysisResultService> listenerServiceClass,
      ExcludedRefs excludedRefs) {
    // 判断install是否在Analyzer进程里，重复执行
    if (isInAnalyzerProcess(application)) {
      return RefWatcher.DISABLED;
    }
    enableDisplayLeakActivity(application);
    // 创建监听器
    HeapDump.Listener heapDumpListener =
        new ServiceHeapDumpListener(application, processType, listenerServiceClass);
    // Refwatcher 监控引用
    RefWatcher refWatcher = androidWatcher(application, heapDumpListener, excludedRefs);
    // ActivityRefWatcher 传入 application 和 refWatcher
    ActivityRefWatcher.installOnIcsPlus(application, refWatcher);

    return refWatcher;
  }
```
如上代码所示，即install方法中，重点关注如下几个方面：

* heapDumpListener 故名思议拿到heapDump交给这个Listener区处理 
* RefWatcher 引用监视器
* ActivityRefWatcher Activity引用监视器

#### 02.ActivityRefWatcher

虽然Refwatcher先出现，但是为了代码阅读的连贯性，先去看一下ActivityRefWatcher这个类：

``` java
public final class ActivityRefWatcher {
  
  @Deprecated
  public static void installOnIcsPlus(Application application, RefWatcher refWatcher) {
    install(application, refWatcher);
  }

  public static void install(Application application, RefWatcher refWatcher) {
    new ActivityRefWatcher(application, refWatcher).watchActivities();
  }

  private final Application.ActivityLifecycleCallbacks lifecycleCallbacks =
      new Application.ActivityLifecycleCallbacks() {
        @Override public void onActivityCreated(Activity activity, Bundle savedInstanceState) {
        }

        @Override public void onActivityStarted(Activity activity) {
        }

        @Override public void onActivityResumed(Activity activity) {
        }

        @Override public void onActivityPaused(Activity activity) {
        }

        @Override public void onActivityStopped(Activity activity) {
        }

        @Override public void onActivitySaveInstanceState(Activity activity, Bundle outState) {
        }

        // 将Activity中的onActivityDestoryed事件注册上去
        @Override public void onActivityDestroyed(Activity activity) {
          ActivityRefWatcher.this.onActivityDestroyed(activity);
        }
      };

  // 将 activity 事件传递到了 refwatcher
  void onActivityDestroyed(Activity activity) {
    // 回调了 refWathcher中的watch方法监控activity
    refWatcher.watch(activity);
  }
}
```
ok，如上代码很清晰的说明了一个作用，通过ActivityLifecycleCallbacks把Activity的ondestory生命周期关联到了ActivityRefWatcher这个上面，并且用refWatcher去监控：

#### 03.RefWatcher

``` java
public final class RefWatcher {

  // 执行内存泄漏检测的 executor
  private final Executor watchExecutor; 
  // 用于查询是否正在调试中，调试中不会执行内存泄漏检测
  private final DebuggerControl debuggerControl; 
  // 用于在判断内存泄露之前，再给一次GC的机会 
  private final GcTrigger gcTrigger; 
  // dump 处内存泄漏的 heap
  private final HeapDumper heapDumper; 
  // 持有那些待检测以及产生内存泄露的引用的key
  private final Set<String> retainedKeys; 
  // 用于判断弱引用所持有的对象是否已被GC
  private final ReferenceQueue<Object> queue; 
  // 用于分析产生的heap文件
  private final HeapDump.Listener heapdumpListener; 
  // 排除一些系统的bug引起的内存泄漏
  private final ExcludedRefs excludedRefs; 

  public RefWatcher(Executor watchExecutor, DebuggerControl debuggerControl, GcTrigger gcTrigger,
      HeapDumper heapDumper, HeapDump.Listener heapdumpListener, ExcludedRefs excludedRefs) {
  }

  //...

  public void watch(Object watchedReference, String referenceName) {
   
    final long watchStartNanoTime = System.nanoTime();
    // 对一个 referenc 添加唯一的一个 key
    String key = UUID.randomUUID().toString();
    retainedKeys.add(key);
    // 将 watch 传入的对象添加一个弱引用
    final KeyedWeakReference reference =
        new KeyedWeakReference(watchedReference, key, referenceName, queue);
    // 在异步线程上开始分析这个弱引用
    watchExecutor.execute(new Runnable() {
      @Override public void run() {
        ensureGone(reference, watchStartNanoTime);
      }
    });
  }
```

#### 04.RefWatcher#ensureGone

ensureGone的作用可以从其名称得出确保Activity是否真的被回收，为什么需要这个确保过程呢？因为最后dump内存信息之前，希望系统已经做了充分的GC，不会存在误判：

``` java
// 避免因为gc不及时带来的误判，leakcanay会进行二次确认进行保证
  void ensureGone(KeyedWeakReference reference, long watchStartNanoTime) {
    long gcStartNanoTime = System.nanoTime();
    // 计算机从调用 watch 到 gc 所用时间
    long watchDurationMs = NANOSECONDS.toMillis(gcStartNanoTime - watchStartNanoTime);
    // 清除此时已经到 RQ 的弱引用
    // 首先调用 removeWeaklyReachableReferences
    // 把已被回收的对象的 key 从 retainedKeys 移除，剩下的 key 都是未被回收的对象
    removeWeaklyReachableReferences();
    // 如果当前的对象已经弱可达，说明不会造成内存泄漏
    if (gone(reference) || debuggerControl.isDebuggerAttached()) {
      return;
    }
    // 如果当前检测对象没有改变其可达状态，则进行手动GC
    gcTrigger.runGc();
    // 再次清除已经到 RQ的弱引用
    removeWeaklyReachableReferences();
    // 如果此时对象还没有到队列，预期被 gc 的对象会出现在队列中，没出现则此时已经可能泄漏
    if (!gone(reference)) {
      long startDumpHeap = System.nanoTime();
      long gcDurationMs = NANOSECONDS.toMillis(startDumpHeap - gcStartNanoTime);
      // dump出来heap，此时认为内存确实已经泄漏了
      File heapDumpFile = heapDumper.dumpHeap();
      if (heapDumpFile == HeapDumper.NO_DUMP) {
        // 如果不能dump出内存则abort
        return;
      }
      long heapDumpDurationMs = NANOSECONDS.toMillis(System.nanoTime() - startDumpHeap);
      // 去分析
      heapdumpListener.analyze(
          new HeapDump(heapDumpFile, reference.key, reference.name, excludedRefs, watchDurationMs,
              gcDurationMs, heapDumpDurationMs));
    }
  }
  //...
}
```

大部分执行逻辑已经在代码中注释的很清楚，这里做个小插曲，分析一下gcTrigger.runGc()：

``` java
public interface GcTrigger {
  GcTrigger DEFAULT = new GcTrigger() {
    @Override public void runGc() {
      // Code taken from AOSP FinalizationTest:
      // https://android.googlesource.com/platform/libcore/+/master/support/src/test/java/libcore/
      // java/lang/ref/FinalizationTester.java
      // System.gc() does not garbage collect every time. Runtime.gc() is
      // more likely to perfom a gc.
      Runtime.getRuntime().gc();
      enqueueReferences();
      System.runFinalization();
    }

    private void enqueueReferences() {
      // Hack. We don't have a programmatic way to wait for the reference queue daemon to move
      // references to the appropriate queues.
      try {
        Thread.sleep(100);
      } catch (InterruptedException e) {
        throw new AssertionError();
      }
    }
  };

  void runGc();
}
```
如上所示，英文也很简单，说的大概意思是‘system.gc()并不会每次都执行，从AOSP中拷贝一段GC的代码’，从而保证能够进行垃圾清理工作。感觉这就是阅读源码带来的好处呀，很多不知道的小技巧。

#### 05. HeapDumper#dumpHeap

继续跟着上述代码，将heap信息dump出来：

``` java
  @Override public File dumpHeap() {
    if (!leakDirectoryProvider.isLeakStorageWritable()) {
      CanaryLog.d("Could not write to leak storage to dump heap.");
      leakDirectoryProvider.requestWritePermissionNotification();
      return NO_DUMP;
    }
    File heapDumpFile = getHeapDumpFile();
    // Atomic way to check for existence & create the file if it doesn't exist.
    // Prevents several processes in the same app to attempt a heapdump at the same time.
    boolean fileCreated;
    try {
      fileCreated = heapDumpFile.createNewFile();
    } catch (IOException e) {
      cleanup();
      CanaryLog.d(e, "Could not check if heap dump file exists");
      return NO_DUMP;
    }

    // 如果没有目录存放，则抛出
    if (!fileCreated) {
      CanaryLog.d("Could not dump heap, previous analysis still is in progress.");
      // Heap analysis in progress, let's not put too much pressure on the device.
      return NO_DUMP;
    }

    FutureResult<Toast> waitingForToast = new FutureResult<>();
    showToast(waitingForToast);

    if (!waitingForToast.wait(5, SECONDS)) {
      CanaryLog.d("Did not dump heap, too much time waiting for Toast.");
      return NO_DUMP;
    }

    Toast toast = waitingForToast.get();
    try {
      Debug.dumpHprofData(heapDumpFile.getAbsolutePath());
      cancelToast(toast);
      return heapDumpFile;
    } catch (Exception e) {
      cleanup();
      CanaryLog.d(e, "Could not perform heap dump");
      // Abort heap dump
      return NO_DUMP;
    }
  }
```

拿到heapdump之后怎么处理呢，接着下面这句

``` java
heapdumpListener.analyze(
          new HeapDump(heapDumpFile, reference.key, reference.name, excludedRefs, watchDurationMs,
              gcDurationMs, heapDumpDurationMs));
```

从源码中跟heapdumpListener发现其为AbstractAnalysisResultService的一个实例，本质上是一个IntentService

``` java 
public abstract class AbstractAnalysisResultService extends IntentService 
```
看看这个抽象类的继承者：

#### 06. HeapAnalyzerService

``` java
public final class HeapAnalyzerService extends IntentService {
  //...
  public static void runAnalysis(Context context, HeapDump heapDump,
      Class<? extends AbstractAnalysisResultService> listenerServiceClass,String processType) {
  //...
  }

  // 重点
  @Override protected void onHandleIntent(Intent intent) {
    if (intent == null) {
      CanaryLog.d("HeapAnalyzerService received a null intent, ignoring.");
      return;
    }
    String listenerClassName = intent.getStringExtra(LISTENER_CLASS_EXTRA);
    HeapDump heapDump = (HeapDump) intent.getSerializableExtra(HEAPDUMP_EXTRA);
    String processType = intent.getStringExtra(PROCESS_TYPE);
    String processName = intent.getStringExtra(PROCESS_NAME);
      HeapAnalyzer heapAnalyzer = new HeapAnalyzer(heapDump.excludedRefs);
      // 使用checkForLeak这个方法来分析内存使用结果
      AnalysisResult result = heapAnalyzer.checkForLeak(heapDump.heapDumpFile, heapDump.referenceKey);
      // 结果回调
      AbstractAnalysisResultService.sendResultToListener(this, listenerClassName, heapDump, result,processType,processName);
  }
}
```
再去看结果回调之前，先去分析一下产生这个result的流程是怎么一个回事：

#### 07. HeapAnalyzer#checkForleak

``` java
 public AnalysisResult checkForLeak(File heapDumpFile, String referenceKey) {
    long analysisStartNanoTime = System.nanoTime();

    if (!heapDumpFile.exists()) {
      Exception exception = new IllegalArgumentException("File does not exist: " + heapDumpFile);
      return failure(exception, since(analysisStartNanoTime));
    }

    try {
      // 将 dump 文件解析成 snapshot 对象，haha库的用法
      HprofBuffer buffer = new MemoryMappedFileBuffer(heapDumpFile);
      HprofParser parser = new HprofParser(buffer);
      Snapshot snapshot = parser.parse();
      deduplicateGcRoots(snapshot);

      Instance leakingRef = findLeakingReference(referenceKey, snapshot);

      // False alarm, weak reference was cleared in between key check and heap dump.
      if (leakingRef == null) {
        return noLeak(since(analysisStartNanoTime));
      }
      // 找到泄漏路径
      return findLeakTrace(analysisStartNanoTime, snapshot, leakingRef);
    } catch (Throwable e) {
      return failure(e, since(analysisStartNanoTime));
    }
  }

```

继续跟findLeakTrace：

``` java
private AnalysisResult findLeakTrace(long analysisStartNanoTime, Snapshot snapshot,
      Instance leakingRef) {

    ShortestPathFinder pathFinder = new ShortestPathFinder(excludedRefs);
    ShortestPathFinder.Result result = pathFinder.findPath(snapshot, leakingRef);

    // False alarm, no strong reference path to GC Roots.
    if (result.leakingNode == null) {
      return noLeak(since(analysisStartNanoTime));
    }

    LeakTrace leakTrace = buildLeakTrace(result.leakingNode);

    String className = leakingRef.getClassObj().getClassName();

    // Side effect: computes retained size.
    snapshot.computeDominators();

    Instance leakingInstance = result.leakingNode.instance;

    long retainedSize = leakingInstance.getTotalRetainedSize();

    retainedSize += computeIgnoredBitmapRetainedSize(snapshot, leakingInstance);

    // 即将跳转到haha这个库去建立最短引用路径
    return leakDetected(result.excludingKnownLeaks, className, leakTrace, retainedSize,
        since(analysisStartNanoTime));
  }
```
上面的leakDetected跟进去发现已经在`leakcanary-analyzer`这个module中了，按照工程结构，这已经属于haha库的用法了，篇幅原因暂且就不跟进去了。

紧接着呢上面的结果回调，找到了AbstractAnalysisResultService这个类：

#### 08.AbstractAnalysisResultService

``` java
public abstract class AbstractAnalysisResultService extends IntentService {


  public static void sendResultToListener(Context context, String listenerServiceClassName,
      HeapDump heapDump, AnalysisResult result,String processType,String processName) {
    Class<?> listenerServiceClass;
    try {
      listenerServiceClass = Class.forName(listenerServiceClassName);
    } catch (ClassNotFoundException e) {
      throw new RuntimeException(e);
    }
    Intent intent = new Intent(context, listenerServiceClass);
    intent.putExtra(HEAP_DUMP_EXTRA, heapDump);
    intent.putExtra(RESULT_EXTRA, result);
    intent.putExtra(PROCESS_TYPE,processType);
    intent.putExtra(PROCESS_NAME,processName);
    context.startService(intent);
  }

  public AbstractAnalysisResultService() {
    super(AbstractAnalysisResultService.class.getName());
  }

  @Override protected final void onHandleIntent(Intent intent) {
    HeapDump heapDump = (HeapDump) intent.getSerializableExtra(HEAP_DUMP_EXTRA);
    AnalysisResult result = (AnalysisResult) intent.getSerializableExtra(RESULT_EXTRA);
    String process =  intent.getStringExtra(PROCESS_TYPE);
    String processName =  intent.getStringExtra(PROCESS_NAME);

    boolean delFile = true;
    try {
      delFile = onHeapAnalyzed(heapDump, result, process, processName);
    } finally {
      //noinspection ResultOfMethodCallIgnored
      if (delFile)
        heapDump.heapDumpFile.delete();
    }
  }

  protected abstract boolean onHeapAnalyzed(HeapDump heapDump, AnalysisResult result, String processType, String processName);
}
```

同样，这也只是一个抽象类，找到他的继承类DisplayLeakService：

#### 09.DisplayLeakService

``` java
public class DisplayLeakService extends AbstractAnalysisResultService {

    @Override
    protected final boolean onHeapAnalyzed(HeapDump heapDump, AnalysisResult result, String processType, String processName) {
        return onCustomHeapAnalyzed(heapDump, result, processType, processName);
        if (PROCESS_CLIENT.equals(processType)) {
            boolean resultSaved = false;
            boolean shouldSaveResult = result.leakFound || result.failure != null;
            if (shouldSaveResult) {
                resultSaved = saveResult(heapDump, result);

                if (result.failure != null) {
                    File resultFile = new File(heapDump.heapDumpFile.getParentFile(), "analysis_fail" + ".bt");
                    saveLeakInfo(heapDump, resultFile, leakInfo);
                }
            } else {
                //无泄露结果
                File resultFile = new File(heapDump.heapDumpFile.getParentFile(), "no_leakinfo" + ".db");
                saveLeakInfo(heapDump, resultFile, leakInfo);

            }

            Intent pendingIntent = null;
            if (resultSaved) {
               // DisplayLeakActivity显示工作
                pendingIntent = DisplayLeakActivity.createIntent(this,
                        heapDump.referenceKey, "","","");
            }

            String key = heapDump.referenceKey;
            afterClientHandling(heapDump, result, pendingIntent, key);
            return false;
        } else {
            // SERCER
            heapDump = renameHeapdump(heapDump);
            afterServerHandling(heapDump);
        }
        return true;
    }

    private boolean saveResult(HeapDump heapDump, AnalysisResult result) {
        File resultFile = new File(heapDump.heapDumpFile.getParentFile(),
                heapDump.heapDumpFile.getName() + ".result");
        FileOutputStream fos = null;
        try {
            fos = new FileOutputStream(resultFile);
            ObjectOutputStream oos = new ObjectOutputStream(fos);
            oos.writeObject(heapDump);
            oos.writeObject(result);
            return true;
        } catch (IOException e) {
            CanaryLog.d(e, "Could not save leak analysis result to disk.");
        } finally {
            if (fos != null) {
                try {
                    fos.close();
                } catch (IOException ignored) {
                }
            }
        }
        return false;
    }
  	// 扩展式函数
    protected void afterDefaultHandling(HeapDump heapDump, AnalysisResult result, String leakInfo, String processType) {
    }

    protected void afterClientHandling(HeapDump heapDump, AnalysisResult result, Intent intent, String key) {
    }

    protected void afterServerHandling(HeapDump heapDump) {
    }

    protected void afterAnalyzed(PushServiceTaskParam pushTaskParam) {

    }
}
```

在上述代码中，通过DisplayLeakActivity去将泄漏路径显示在应用上面:

``` java
pendingIntent = DisplayLeakActivity.createIntent(this,
                        heapDump.referenceKey, "","","");
```

另外可以复写afterClientHandling去做一些额外的处理，例如将leak信息截取、上传等操作

### 一些思考

了解了LeakCanary的原理之后，发现其实它就是在对象不可用的时候去判断对象是否被回收了，但leakcanary只检查了Activity，我们是否可以检查其他对象呢，毕竟Activity泄漏只是内存泄漏的一种，答案当然是可以的，我们只要需要进行如下操作:

``` java
LeakCanary.install(app).watch(object)
```

但调用这个方法有个前提就是，我们在调用这个方法的时候确定了这个object已经不需要了，可以被回收了才能调用这个方法，通过这种方式我们就可以对任何对象都进行检测了。

LeakCanary有个值得注意的地方，通过下面这个函数来判断进程是否在后台：

``` java
 public static boolean isInServiceProcess(Context context, Class<? extends Service> serviceClass) {
    PackageManager packageManager = context.getPackageManager();
    PackageInfo packageInfo;
    try {
      packageInfo = packageManager.getPackageInfo(context.getPackageName(), GET_SERVICES);
    } catch (Exception e) {
      CanaryLog.d(e, "Could not get package info for %s", context.getPackageName());
      return false;
    }
    String mainProcess = packageInfo.applicationInfo.processName;

    ComponentName component = new ComponentName(context, serviceClass);
    ServiceInfo serviceInfo;
    try {
      serviceInfo = packageManager.getServiceInfo(component, 0);
    } catch (PackageManager.NameNotFoundException ignored) {
      // Service is disabled.
      return false;
    }
	// 注意这里
    if (serviceInfo.processName.equals(mainProcess)) {
      CanaryLog.d("Did not expect service %s to run in main process %s", serviceClass, mainProcess);
      // Technically we are in the service process, but we're not in the service dedicated process.
      return false;
    }

    // 为PID找到对应的Process
    int myPid = android.os.Process.myPid();
    ActivityManager activityManager =
        (ActivityManager) context.getSystemService(Context.ACTIVITY_SERVICE);
    ActivityManager.RunningAppProcessInfo myProcess = null;
    List<ActivityManager.RunningAppProcessInfo> runningProcesses =
        activityManager.getRunningAppProcesses();
    if (runningProcesses != null) {
      for (ActivityManager.RunningAppProcessInfo process : runningProcesses) {
        if (process.pid == myPid) {
          myProcess = process;
          break;
        }
      }
    }
    if (myProcess == null) {
      CanaryLog.d("Could not find running process for %d", myPid);
      return false;
    }

    return myProcess.processName.equals(serviceInfo.processName);
  }
```



好了，以上就是Leakcanary的工作流程以及原理。大道至简，细节入微。加油！！！
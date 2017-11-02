---
layout: post
title: 深入理解 Java 之 FutureTask 和 Callable
date: 2017-09-02 17:34:19 +0800
categories: 
---

在阅读AsyncTask源码时候，看到FutureTask的概念，当时只知道它与线程创建有关系。今天来探究一下它的源码结构。
同时也有一些面试题，询问Java中创建线程的几种方式，除了Thread和Runnable之外，提到了Callable的概念。本文重点分析一下这两个概念的用法以及与平时创建线程方式的区别。


### 基本使用

先看一下使用的实例Demo，代码如下所示：

```java
public class FutureTaskDemo {
    
    public static void main(String[] args) {
        // 创建一个ExecutorService对象
        ExecutorService executor = Executors.newCachedThreadPool();
        // new 一个Callable实例
        Task task = new Task();
        // new一个
        FutureTask<Integer> futureTask = new FutureTask<Integer>(task);
        // 提交futureTask对象进入线程池
        executor.submit(futureTask);
        // 关闭线程池
        executor.shutdown();

        try {
            Thread.sleep(1000);
        } catch (InterruptedException e1) {
            e1.printStackTrace();
        }

        System.out.println("主线程在执行任务");

        try {
            // 获取futuretask结果
            System.out.println("task运行结果" + futureTask.get());
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }

        System.out.println("所有任务执行完毕");
    }
}
```

同时定义一个实现Callable接口的Task类如下所示：

``` java
public static class Task implements Callable<Integer> {
        @Override
        public Integer call() throws Exception {
            System.out.println("子线程在进行计算");
            Thread.sleep(3000);
            int sum = 0;
            for (int i = 0; i < 100; i++)
                sum += i;
            return sum;
        }
    }
```

结果输出：

``` shell
子线程在进行计算
主线程在执行任务
task运行结果4950
所有任务执行完毕
```

从FutureTask用法以及返回的结果可见FutureTask是可以去执行Callable的，并且Callable独立的线程可以返回自己的执行结果。

### Callable 

跟着IDE中的源码进入到Callable的声明中了解一下其结构：

``` java
@FunctionalInterface
public interface Callable<V> {
    /**
     * Computes a result, or throws an exception if unable to do so.
     *
     * @return computed result
     * @throws Exception if unable to compute a result
     */
    V call() throws Exception;
}
```

摘要处其文档注释如下：

> A task that returns a result and may throw an exception.
>  Implementors define a single method with no arguments called
>  {@code call}.
>  <p>The {@code Callable} interface is similar to {@link
>  java.lang.Runnable}, in that both are designed for classes whose
>  instances are potentially executed by another thread.  A
>  {@code Runnable}, however, does not return a result and cannot
>  throw a checked exception.


### Runnable

点进去看一下Runnable接口的定义：

``` java
 @FunctionalInterface
public interface Runnable {
    public abstract void run();
}
```

综合上述简单来讲Callable与Runnable作用一样，区别在于Callable有返回值，并且出现异常能抛出来。

### FutureTask

接下来重点看一下示例中task的包装类FutureTask。如下所示发现RutureTask是实现RunnableFuture接口的一个类：

``` java
public class FutureTask<V> implements RunnableFuture<V> 
```

### RunnableFuture

再去看看RunnableFuture的结构：

``` java
public interface RunnableFuture<V> extends Runnable, Future<V> {
    /**
     * Sets this Future to the result of its computation
     * unless it has been cancelled.
     */
    void run();
}
```

发现 RunnableFuture接口继承自Runnable和Future接口，接下来重点关注Future接口。

### Future

Future根据源代码解释其作用是对Callable或者Runnable进行管理，取消、检测完成与否获取最终结果等。

``` java
public interface Future<V> {

    boolean cancel(boolean mayInterruptIfRunning);
    boolean isCancelled();
    boolean isDone();
    V get() throws InterruptedException, ExecutionException;
    V get(long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}
```

* cancel方法用来取消任务，如果取消任务成功则返回true，如果取消任务失败则返回false。参数mayInterruptIfRunning表示是否允许取消正在执行却没有执行完毕的任务，如果设置true，则表示可以取消正在执行过程中的任务。如果任务已经完成，则无论mayInterruptIfRunning为true还是false，此方法肯定返回false，即如果取消已经完成的任务会返回false；如果任务正在执行，若mayInterruptIfRunning设置为true，则返回true，若mayInterruptIfRunning设置为false，则返回false；如果任务还没有执行，则无论mayInterruptIfRunning为true还是false，肯定返回true。

* isCancelled方法表示任务是否被取消成功，如果在任务正常完成前被取消成功，则返回 true。

* isDone方法表示任务是否已经完成，若任务完成，则返回true；

* get()方法用来获取执行结果，这个方法会产生阻塞，会一直等到任务执行完毕才返回；

* get(long timeout, TimeUnit unit)用来获取执行结果，如果在指定时间内，还没获取到结果，就直接返回null。

在本文代码中示例中就通过：

``` java
 futureTask.get()
```
获取callable的执行结果。

OK，通过上述示例Demo的讲解，相信大家都应该明白了Callable与FutureTask的用法。
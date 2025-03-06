---
title: 【Java】线程池
date: 2021-03-15 16:06 +0800
categories: [Java]
tags: [java, thread pool, concurrency]
---
## 1.线程池的主要作用
* 不同请求之间重复利用线程，无需频繁地创建和销毁线程，降低系统开销。
* 控制线程数量上限，避免创建过多的线程耗尽进程内存空间，同时减少线程上下文切换次数。

## 2.ThreadPoolExecutor类

![线程池类图](/assets/images/java-thread-pool/线程池类图.png)

线程池由两个核心数据结构组成：
* 线程集合`workers`：存放执行任务的线程，是一个`HashSet<Worker>`。
* 任务等待队列`workQueue`：存放等待线程池调度执行的任务，是一个阻塞队列`BlockingQueue<Runnable>`（阻塞队列的实现方式：Lock+Condition）。

![线程池核心数据结构](/assets/images/java-thread-pool/线程池核心数据结构.png)

### 2.1 线程池的核心参数

| 参数 | 用途 |
| --- | --- |
| `corePoolSize` | 核心线程数：工作线程数少于这个数量时会创建新的线程 |
| `maximumPoolSize` | 最大线程数：工作线程数达到这个数量时不再创建新的线程 |
| `keepAliveTime` | 空闲线程最大存活时间 |

### 2.2 创建线程池
使用`Executors`类的工厂方法创建不同类型的线程池：

| 方法 | 说明 |
| --- | --- |
| `newFixedThreadPool(n)` | 创建固定线程数量的线程池(`corePoolSize = maximumPoolSize = n`) |
| `newSingleThreadExecutor()` | 创建单个线程的线程池(`corePoolSize = maximumPoolSize = 1`) |
| `newCachedThreadPool()` | 线程池会按需创建新线程，但优先复用已有的线程(`corePoolSize = 0, maximumPoolSize = Integer.MAX_VALUE`) |

### 2.3 任务执行流程
任务由`execute()`方法提交到线程池中调度，在提交任务时会有下面几种场景：
1. 线程数量<`corePoolSize`：此时任务不会进等待队列，线程池直接创建一个线程执行提交的任务。
2. `corePoolSize`<=线程数量<`maximumPoolSize`且等待队列未满：任务直接添加到等待队列，等待线程池调度执行。
3. `corePoolSize`<=线程数量<`maximumPoolSize`且等待队列已满：线程池会新创建一个线程执行提交的任务。
4. 线程数量>=`maximumPoolSize`且等待队列已满：线程池会拒绝任务，执行一个指定的拒绝策略。
5. 线程池已关闭：拒绝任务，执行一个指定的拒绝策略。

线程创建之后，会不停从等待队列`workQueue`中拉取任务（`ThreadPoolExecutor.getTask()`方法），`workQueue`是一个线程安全的阻塞队列，所以不存在线程安全问题，拉取到任务之后执行任务。

拉取任务时有两种情况：
* 线程池设置了`keepAliveTime`参数，并且此时线程池中的线程数量超过`corePoolSize`，则从队列中拉取任务时会设置`keepAliveTime`为超时时间，超过这个时间之后，该线程不再等待任务，直接跑完`run()`方法体，线程被回收。
* 否则线程会无限等待任务队列直到有任务到来。

### 2.4 拒绝策略
当线程集合和等待队列都满时线程无法调度任务，这时线程池会执行一个拒绝策略（`RejectedExecutionHandler`接口，在`ThreadPoolExecutor.reject()`方法中调用）。

JDK内置的拒绝策略有以下几种（都是`ThreadPoolExecutor`的内部类）：
* `CallerRunsPolicy`：调用线程自己执行任务
* `AbortPolicy`：抛出`RejectedExecutionException`异常
* `DiscardPolicy`：直接丢弃任务（什么都不做）
* `DiscardOldestPolicy`：丢弃等待队列中最老的任务，然后重新提交当前任务

### 2.5 异常处理
默认情况下线程池的`runWorker()`方法会**捕获任务抛出的所有异常，但不做任何处理**。

`runWorker()`方法的核心代码如下：

```java
beforeExecute(wt, task);
Throwable thrown = null;
try {
    task.run();
} catch (RuntimeException x) {
    thrown = x; throw x;
} catch (Error x) {
    thrown = x; throw x;
} catch (Throwable x) {
    thrown = x; throw new Error(x);
} finally {
    afterExecute(task, thrown);
}
```

其中`beforeExecute()`和`afterExecute()`的默认实现什么都不做（这两个方法都是`protected`）。

存在的问题：这种处理方式能够保证任务抛出的异常不会影响工作线程和其他任务的执行，但也不会打印日志，无法查看报错信息。

解决方法：

（1）在提交的任务中捕获并处理异常

```java
public class MyTask implements Runnable {
    @Override
    public void run() {
        try {
            // 业务逻辑
        }
        catch (Throwable e) {
            // 处理异常
        }
        finally {
            // 其他处理
        }
    }
}
```

（2）自定义线程池：继承`ThreadPoolExecutor`类并实现`afterExecute()`方法，在该方法中处理异常。

```java
public class MyThreadPoolExecutor extends ThreadPoolExecutor {
    // 构造器

    @Override
    protected void afterExecute(Runnable r, Throwable t) {
        super.afterExecute(r, t);
        if (t != null) {
            // 处理异常
        }
    }
}
```

（3）使用线程的异常处理器
* 实现`Thread.UncaughtExceptionHandler`接口，在`uncaughtException()`方法中处理异常。
* 实现`ThreadFactory`接口，调用线程的`setUncaughtExceptionHandler()`方法设置新线程的异常处理器。
* 将`ThreadFactory`对象传递给`ThreadPoolExecutor`的构造器或`Executors`类的静态方法的参数。

注：线程的异常处理器由JVM调用。

```java
public class MyExceptionHandler implements Thread.UncaughtExceptionHandler {
    @Override
    public void uncaughtException(Thread t, Throwable e) {
        // 处理异常
    }
}

public class MyThreadFactory implements ThreadFactory {
    private static final ThreadFactory defaultFactory = Executors.defaultThreadFactory();
    private final Thread.UncaughtExceptionHandler handler = new MyExceptionHandler();

      @Override
      public Thread newThread(Runnable r) {
          Thread thread = defaultFactory.newThread(r);
          thread.setUncaughtExceptionHandler(handler);
          return thread;
      }
}

public class Main {
    public static void main(String[] args) {
        ExecutorService pool = Executors.newFixedThreadPool(10, new MyThreadFactory());
        pool.execute(new MyTask());
    }
}
```

（4）继承`ThreadGroup`类并覆盖`uncaughtException()`方法，将`ThreadGroup`对象传递给`Thread`的构造器参数。

（5）使用`Future`
* 如果使用`submit()`方法提交任务则会返回一个`Future`对象，返回结果及异常都可以通过`Future`对象获取。
* `Future.get()`方法：如果任务正常结束则返回结果，如果任务抛出异常则抛出`ExecutionException`异常。

```java
ExecutorService pool = Executors.newFixedThreadPool(10);
Future<?> future = pool.submit(new MyTask());
try {
    future.get();
}
catch (InterruptedException e) {
    Thread.currentThread().interrupt();
}
catch (ExecutionException e) {
    Throwable exception = e.getCause();
    // 处理异常
}
```

注：前三种方式仅适用于通过`execute()`提交的任务，通过`submit()`提交的任务应使用第五种方式。

## 参考
<https://baijiahao.baidu.com/s?id=1641469444994560637&wfr=spider&for=pc>

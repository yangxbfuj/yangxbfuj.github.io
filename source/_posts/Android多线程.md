---
title: Android多线程
date: 2017-02-26 14:16:28
tags:
categories:
- Android
- 线程
---

相关项目地址 https://github.com/yangxbfuj/AndroidThreadDemo

本文介绍一些线程控制相关的内容，不一定依赖于 Android 系统。

<!-- more -->

### 多线程 -- Thread 和 Runnable

Thread 实际上就是一个 Runnable 实现。

```java
public class Thread implements Runnable{
    
    // 省略其它代码
    
    public synchronized void start() {
        /**
         * This method is not invoked for the main method thread or "system"
         * group threads created/set up by the VM. Any new functionality added
         * to this method in the future may have to also be added to the VM.
         *
         * A zero status value corresponds to state "NEW".
         */
        if (threadStatus != 0)
            throw new IllegalThreadStateException();

        /* Notify the group that this thread is about to be started
         * so that it can be added to the group's list of threads
         * and the group's unstarted count can be decremented. */
        group.add(this);

        started = false;
        try {
            nativeCreate(this, stackSize, daemon); // 开启线程
            started = true;
        } finally {
            try {
                if (!started) {
                    group.threadStartFailed(this);
                }
            } catch (Throwable ignore) {
                /* do nothing. If start0 threw a Throwable then
                  it will be passed up the call stack */
            }
        }
    }
}
```

Thread 实际上也是执行 Runnable 声明的 run() 函数

### 线程 wait 、 sleep 、join 和 jield

用于线程控制，同步等。
wait() : 线程进入到和该对象相关的等待池中，同时失去了当前对象的同步锁，是的其它线程可以访问当前对象。用户可以使用 notify、notifyAll或者制定睡眠时间来唤醒当前等待池中的线程。也就是进入了阻塞队列。
    PS：wait()、notify()、notifyAll()必须放在 synchronized block 中，否者会抛出异常。
sleep  : Thread 的静态函数，并不释放同步锁。也就是强制放入就绪队列XXX时间。
join   : 等待目标线程执行完成之后再继续执行。
yield  : 线程礼让。

### 与多线程相关的方法 -- Callable、Future和FutureTask

与 Runnable（ Thread 本质也是 Runnable） 不同的是，这几个只能使用在线程池中。
重要的事情说3遍: 线程池！线程池！线程池！

#### Callable

类似于 Runnable ，但是 Callable 可以返回一个范型的结果。

```java
public interface Callable<V>{
    V call() throws Exception;
} 
```

#### Future

相比于 Callable 和 Runnable 不同之处在于 Future 为线程池制定了一个可管理的任务标准。

```java
public interface Future<V>{
    boolean cancel(boolean mayInterruptRunning);
    // 查询该任务是否已经取消
    boolean isCancelled();

    // 判断是否已经完成
    boolean isDone();

    // 获取结果，如果任务未完成，则阻塞调用线程，直到完成
    V get() throws InterruptedException,ExecutionException;

    // 获取结果，如果任务还未完成，则阻塞最多阻塞相应时间
    V get(long timeout,TimeUnit unit) throws InterruptedException,ExecutionException,TimeoutException;
}
```

#### FutureTask

其实就是 Future 的实现类。

```java
public class FutureTask<V> implements RunnableFuture<V> 
```

```java
public interface RunnableFuture<V> extends Runnable, Future<V> 
```

FutureTask 会像 Thread 包装 Runnable 那样对 Runnable 和Callable<V> 进行包装。而在包装过程中，Runnabe 会被转换为 Callable。

由于 FutureTask 实现了 Runnable ,所以 FutureTask 可以用于 Thread ，也可以用于线程池。而且还可以通过 get() 返回结果。

#### 线程池

##### 线程池的优点：
1.重用存在的线程，减少对象的创建、销毁的开销。TCB 管理。
2.可以有效的控制最大并发线程数，提高系统资源的使用率，同时避免过多的资源竞争，避免堵塞。
3.提供定时执行、定期执行、单线程、并发数控制等功能。

线程池都实现了 ExecutorService 接口。

不想写了，自己去查资料吧。
还有一些内容，开新的文章。




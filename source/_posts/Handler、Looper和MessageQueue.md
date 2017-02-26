---
title: Handler、Looper 和 MessageQueue
date: 2017-02-26 13:11:03
tags:
categories:
- Andriod
---

相关学习项目地址：https://github.com/yangxbfuj/AndroidThreadDemo

此文简单介绍了一些Android多线程中 Hanlder 、Looper 和 MessageQueue等的内容。

<!-- more -->

## Andriod 中的消息机制

### Handler 、 Looper 和 MessageQueue
    
UI线程会关联一个消息队列，所有的操作都会被封装成消息然后交给主线程处理。为了保证主线程不会主动退出，会将获取消息的操作放在一个死循环中，
样程序就相当于一直执行在死循环中。（当然在循环中，线程会在没有消息时发生阻塞）。UI消息循环在 ActivityThread.main 方法中创建的。

#### 系统如何将消息投递到消息队列？系统如何从消息队列中获得消息？ 

每个 Handler 都会关联一个消息队列，消息队列被封装到一个 Looper 中，而每个 Looper 又会关联一个线程（ Looper 封装在 ThreadLocal 中），最终就等于没个消息队列会关联一个线程。Handler 就是一个消息处理器，将消息投递给消息队列，然后再有对应的线程从消息队列中出个取出消息，并且执行。

```java
public Handler(Callback callback, boolean async) {
        if (FIND_POTENTIAL_LEAKS) {
            final Class<? extends Handler> klass = getClass();
            if ((klass.isAnonymousClass() || klass.isMemberClass() || klass.isLocalClass()) &&
                    (klass.getModifiers() & Modifier.STATIC) == 0) {
                Log.w(TAG, "The following Handler class should be static or leaks might occur: " +
                    klass.getCanonicalName());
            }
        }

        mLooper = Looper.myLooper();//此处Handler获得了响应的Looper对象
        if (mLooper == null) {
            throw new RuntimeException(
                "Can't create handler inside thread that has not called Looper.prepare()");
        }
        mQueue = mLooper.mQueue; // 获取消息队列
        mCallback = callback;
        mAsynchronous = async;
    }
```
根据 Handler 的构造函数可知，在 Handler 创建时就获得了当前线程的 Looper 对象。

然后继续跟踪 Looper 对象的来源:

```java
static final ThreadLocal<Looper> sThreadLocal = new ThreadLocal<Looper>();

public static @Nullable Looper myLooper() {
    return sThreadLocal.get(); // 从线程中取得的
}
```

```java
public T get() {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null)
            return (T)e.value;
        }
    return setInitialValue();
}
```
```java
/**
 * Initialize the current thread as a looper, marking it as an
 * application's main looper. The main looper for your application
 * is created by the Android environment, so you should never need
 * to call this function yourself.  See also: {@link #prepare()}
 */
public static void prepareMainLooper() {
    prepare(false);
    synchronized (Looper.class) {
        if (sMainLooper != null) {
            throw new IllegalStateException("The main Looper has already been prepared.");
        }
        sMainLooper = myLooper();
    }
}

private static void prepare(boolean quitAllowed) {
    if (sThreadLocal.get() != null) {
        throw new RuntimeException("Only one Looper may be created per thread");
    }
    sThreadLocal.set(new Looper(quitAllowed));
}
```

#### 为什么更新UI线程的 Handler 必须在UI线程中创建？

sThreadLocal.get() 中调用了 Thread t = Thread.currentThread() 然后再通过 t 获取到了 Looper ，所以更新UI线程的 Handler 必须在UI线程中创建。同理推导至其它派生线程。

#### 消息循环又是如何执行的呢？

答案在 Looper 中。

```java
/**
 * Run the message queue in this thread. Be sure to call
 * {@link #quit()} to end the loop.
 */
public static void loop() {
    final Looper me = myLooper();
    if (me == null) {
        throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
    }
    final MessageQueue queue = me.mQueue;

    // Make sure the identity of this thread is that of the local process,
    // and keep track of what that identity token actually is.
    Binder.clearCallingIdentity();
    final long ident = Binder.clearCallingIdentity();

    for (;;) {
        Message msg = queue.next(); // might block 此处在对立中没有消息时，阻塞线程。
        if (msg == null) {
            // No message indicates that the message queue is quitting.
            return;
        }

        // This must be in a local variable, in case a UI event sets the logger
        final Printer logging = me.mLogging;
        if (logging != null) {
            logging.println(">>>>> Dispatching to " + msg.target + " " +
                    msg.callback + ": " + msg.what);
        }

        final long traceTag = me.mTraceTag;
        if (traceTag != 0 && Trace.isTagEnabled(traceTag)) {
            Trace.traceBegin(traceTag, msg.target.getTraceName(msg));
        }
        try {
            msg.target.dispatchMessage(msg);  // 将消息分配给 Handler
        } finally {
            if (traceTag != 0) {
                Trace.traceEnd(traceTag);
            }
        }

        if (logging != null) {
            logging.println("<<<<< Finished to " + msg.target + " " + msg.callback);
        }

        // Make sure that during the course of dispatching the
        // identity of the thread wasn't corrupted.
        final long newIdent = Binder.clearCallingIdentity();
        if (ident != newIdent) {
            Log.wtf(TAG, "Thread identity changed from 0x"
                    + Long.toHexString(ident) + " to 0x"
                    + Long.toHexString(newIdent) + " while dispatching to "
                    + msg.target.getClass().getName() + " "
                    + msg.callback + " what=" + msg.what);
        }

        msg.recycleUnchecked(); // 回收消息
    }
}
```
Looper 对象通过从 MessageQueue 中取得响应的 Message ,然后将 Message 调用痛过 Message 的 target （实质就是一个 Handler ）属性调用。

在来看看 Handler 是如何处理消息的：

```java
/**
 * Handle system messages here.
 */
public void dispatchMessage(Message msg) {
    if (msg.callback != null) {
        handleCallback(msg);
    } else {
        if (mCallback != null) {
            if (mCallback.handleMessage(msg)) {
                return;
            }
        }
        handleMessage(msg);
    }
}

private static void handleCallback(Message message) {
    message.callback.run();
}

/**
 * Subclasses must implement this to receive messages.
 * 用户重写
 */
public void handleMessage(Message msg) {
}
    
```

#### 在子线程中创建 Handler 为什么会抛出异常？

根据 Handler 初始化方法，Handler 需要获取 Looper.myLooper() ,而新线程的 Looper.myLooper() 的结果为空，也就是说新线程并没有创建相关的 Looper （自然包括 MessageQueue）。

那么如何解决呢？答案是在新线程中调用 Looper.prepare()方法，创建一个 Looper对象。

```java
/** Initialize the current thread as a looper.
 * This gives you a chance to create handlers that then reference
 * this looper, before actually starting the loop. Be sure to call
 * {@link #loop()} after calling this method, and end it by calling
 * {@link #quit()}.
 */
public static void prepare() {
    prepare(true);
}

private static void prepare(boolean quitAllowed) {
    if (sThreadLocal.get() != null) {
        throw new RuntimeException("Only one Looper may be created per thread");
    }
    sThreadLocal.set(new Looper(quitAllowed)); // 创建一个 Looper 对象
}
```






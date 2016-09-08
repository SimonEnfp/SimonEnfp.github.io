---
title: Android Handler工作原理及源码学习
date: 2016-09-01 16:15:28
tags:
categories: 学习笔记
---

一般使用方法是先在主线程创建一个Handler，等待接收消息

```python
    Handler handler = new Handler(new Handler.Callback()     {
    @Override
    public boolean handleMessage(Message msg) {
        return true;
    }
});
```

通过Handler构造方法可以知道在某个线程new Handler之前必须要为此线程创一个Looper否则会报错
从Handler构造函数部分代码可以看出：
```python
public Handler(Callback callback, boolean async) {    
    mLooper = Looper.myLooper();
    if (mLooper == null) {
        throw new RuntimeException(
            "Can't create handler inside thread that has not called Looper.prepare()");
    }
    mQueue = mLooper.mQueue;
    mCallback = callback;
    mAsynchronous = async;
}
```


由于主线程已经创建一个Looper因此可以直接创建Handler，在ActvityThread的main方法中，
```python
public static void prepareMainLooper() {
    prepare(false);
    synchronized (Looper.class) {
        if (sMainLooper != null) {
            throw new IllegalStateException("The main Looper has already been prepared.");
        }
        sMainLooper = myLooper();
    }
}
```
prepare会new一个Looper并通过sThreadLocal把此Looper保存到当前线程中去，如下：
```python
private static void prepare(boolean quitAllowed) {
    if (sThreadLocal.get() != null) {
        throw new RuntimeException("Only one Looper may be created per thread");
    }
    sThreadLocal.set(new Looper(quitAllowed));
}
```
看到new Looper方法：
```python
private Looper(boolean quitAllowed) {
    mQueue = new MessageQueue(quitAllowed);
    mThread = Thread.currentThread();
}
```
这里也为Looper初始化了一个MessageQueue
sThreadLocal.set方法将此Looper保存到主线程的ThreadLocal.Values中去，以便查找指定线程的Looper
```python
public void set(T value) {
    Thread currentThread = Thread.currentThread();
    Values values = values(currentThread);
    if (values == null) {
        values = initializeValues(currentThread);
    }
    values.put(this, value);
}
```
ThreadLocal是一个线程内部的数据存储类，就是在指定的线程存储和读取数据，以线程为区分，每个线程互不干涉
在Looper中的ThreadLocal是静态final的
```python
static final ThreadLocal<Looper> sThreadLocal = new ThreadLocal<Looper>();
```
这样就可以在不同的线程中访问同一个ThreadLocal对象，但是它们获取到的值却可以不一样，互相独立，因为每个线程创建它的Looper的时候都会用同一个ThreadLocal对象把Looper存储到自己的ThreadLocal.Values中去，这样每次获取的时候无论是在哪个线程都会通过同一个ThreadLocal对象到自己的ThreadLocal.Values里去获取。

有了前面的储备工作，主线程有了Looper，因此接下来可以在主线程中创建handler
创建handler的时候会调用下面这句来获取Looper
mLooper = Looper.myLooper();通过这句话获取当前线程Looper
其调用Looper的方法：
```python
public static @Nullable Looper myLooper() {
    return sThreadLocal.get();
}
```
调用ThreadLocal中get方法
```python
public T get() {
    // Optimized for the fast path.
    Thread currentThread = Thread.currentThread();
    Values values = values(currentThread);
    if (values != null) {
        Object[] table = values.table;
        int index = hash & values.mask;
        if (this.reference == table[index]) {
            return (T) table[index + 1];
        }
    } else {
        values = initializeValues(currentThread);
    }

    return (T) values.getAfterMiss(this);
}
```

刚刚创建Looper的时候已经通过sThreadLocal的set方法把Looper存储到对应线程的Values里了，看来为的就是此刻

```python
public Handler(Callback callback, boolean async) {
    mLooper = Looper.myLooper();
    if (mLooper == null) {
        throw new RuntimeException(
            "Can't create handler inside thread that has not called Looper.prepare()");
    }
    mQueue = mLooper.mQueue;
    mCallback = callback;
    mAsynchronous = async;
}
```
可以知道此时的Handler拿到了当前线程的Looper和MessageQueue并初始化成员变量
此时如果在子线程中发消息：
    
    handler.sendMessage(message);
在Handler中最终会调用这个方法：
```python
private boolean enqueueMessage(MessageQueue queue, Message msg, long uptimeMillis) {
    msg.target = this;
    if (mAsynchronous) {
        msg.setAsynchronous(true);
    }
    return queue.enqueueMessage(msg, uptimeMillis);
}
```
这里的MessageQueue就是创建Handler的所在线程的Looper的MessageQueue，调用它的
enqueueMessage方法，其实就是往消息队列里插入一条消息，主要通过无限循环的next方法来从消息队列中获取新消息，当有新消息到来时，next方法会返回这条消息并从单链表移除
Looper的loop方法：
```python
for (;;) {
    Message msg = queue.next(); // might block
    if (msg == null) {
        // No message indicates that the message queue is quitting.
        return;
    }

    // This must be in a local variable, in case a UI event sets the logger
    Printer logging = me.mLogging;
    if (logging != null) {
        logging.println(">>>>> Dispatching to " + msg.target + " " +
                msg.callback + ": " + msg.what);
    }

    msg.target.dispatchMessage(msg);

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

    msg.recycleUnchecked();
}
```
可以发现，loop也是一个死循环，loop方法会调用MessageQueue的next方法来获取新消息，如果next返回了新消息，Looper就会处理这条消息：
msg.target.dispatchMessage(msg);
这里的msg.target是发送这条消息的Handler对象，这样Handler发送的消息最终又交给它的dispatchMessage来处理了，
```python
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
```
到这里就很眼熟了，mCallback就是在主线程中创建Handler时传入的Callback，这样就成功将代码逻辑从子线程切换到主线程执行了

*以上仅仅是自己学习过程中总结的笔记*


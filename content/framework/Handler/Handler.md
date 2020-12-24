---
title: 'Handler'
date: 2018-11-28T15:14:39+10:00
weight: 2
summary: "从handler的面试题出发了解handler"
---

> handler 是学习中问得最常见的一个问题。而这一次我将从几个handler最常见的几个问题来开始学习这个问题

- 一个线程可以有几个Handler
- 一个线程可以有几个looper
- handler的内存泄漏的原因？为什么其他的内部类没有提说过有这个问题？
- 为何主线程可以new Handler？ 如果想要在子线程中new handler要做些什么准备？
- 子线程中维护的Looper，消息队列无消息的时候的处理方案是什么？有什么用？
- 既然可以存在多个Handler 往MessageQueue中add 数据，那内部是如何确保线程数据安全的？
- 我们使用Message 时应该如何创建它？
- 使用Handler的postDelay后消息队列会有什么变化？
- looper死循环为什么不会导致应用卡死？


## 那么我将会从源码的角度上开始分析这几个问题。


>[开篇](https://www.jianshu.com/p/aa17a07aaae7)
> [handler在线源码](http://androidxref.com/9.0.0_r3/xref/frameworks/base/core/java/android/os/Handler.java)

 先来看看我们的最常见的使用handler的方式
```java
        Handler handler = new Handler() {
            @Override
            public void handleMessage(Message msg) {
                super.handleMessage(msg);
                switch (msg.what) {
                    case 1:
                        break;
                }
            }
        };

        handler.sendEmptyMessage(1);
```
然后我们从源码中分析，所有的 send post 方法最终都会调用到 `enqueueMessage` 

![handler外部调用关系图](https://upload-images.jianshu.io/upload_images/12975041-6be0ac83b80509fc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```java

    private boolean enqueueMessage(MessageQueue queue, Message msg, long uptimeMillis) {
        msg.target = this;
        if (mAsynchronous) {
            msg.setAsynchronous(true);
        }
        return queue.enqueueMessage(msg, uptimeMillis);
    }

```
那么我们下一步就要探寻queue.enqueueMessage 里面做了什么。这一步我们看到 queue来自于 [Looper](http://androidxref.com/9.0.0_r3/xref/frameworks/base/core/java/android/os/Looper.java)
```java
// Handler.java
        mQueue = mLooper.mQueue;//
```
```java
// Looper.java
        mQueue = new MessageQueue(quitAllowed);//
```
[MessageQueue.java](http://androidxref.com/9.0.0_r3/xref/frameworks/base/core/java/android/os/MessageQueue.java)
```java

    boolean enqueueMessage(Message msg, long when) {
        if (msg.target == null) {
            throw new IllegalArgumentException("Message must have a target.");
        }
        if (msg.isInUse()) {
            throw new IllegalStateException(msg + " This message is already in use.");
        }

        synchronized (this) {
            if (mQuitting) {
                IllegalStateException e = new IllegalStateException(
                        msg.target + " sending message to a Handler on a dead thread");
                Log.w(TAG, e.getMessage(), e);
                msg.recycle();
                return false;
            }

            msg.markInUse();
            msg.when = when;
            Message p = mMessages;
            boolean needWake;
            if (p == null || when == 0 || when < p.when) {
                // New head, wake up the event queue if blocked.
                msg.next = p;
                mMessages = msg;
                needWake = mBlocked;
            } else {
                // Inserted within the middle of the queue.  Usually we don't have to wake
                // up the event queue unless there is a barrier at the head of the queue
                // and the message is the earliest asynchronous message in the queue.
                needWake = mBlocked && p.target == null && msg.isAsynchronous();
                Message prev;
                for (;;) {
                    prev = p;
                    p = p.next;
                    if (p == null || when < p.when) {
                        break;
                    }
                    if (needWake && p.isAsynchronous()) {
                        needWake = false;
                    }
                }
                msg.next = p; // invariant: p == prev.next
                prev.next = msg;
            }

            // We can assume mPtr != 0 because mQuitting is false.
            if (needWake) {
                nativeWake(mPtr);
            }
        }
        return true;
    }


    private native void nativePollOnce(long ptr, int timeoutMillis); /*non-static for callbacks*/
    private native static void nativeWake(long ptr);

```
从里面我们可以看到 equeueMessage 只是将message 插入到队列中，至于怎么插入的我们后面再说。
在插入之后这个message 会怎么执行呢，重点在执行`nativeWake(mPtr);` 这一句上。
[课外阅读native 代码地址 android_os_MessageQueue.cpp](http://androidxref.com/9.0.0_r3/xref/frameworks/base/core/jni/android_os_MessageQueue.cpp)

在这里会做一个唤醒，既然有唤醒，就会有沉睡，那么我们来看messagequeue 在哪里调用沉睡的方法`nativePollOnce`
```java

    Message next() {
        // Return here if the message loop has already quit and been disposed.
        // This can happen if the application tries to restart a looper after quit
        // which is not supported.
        final long ptr = mPtr;
        if (ptr == 0) {
            return null;
        }

        int pendingIdleHandlerCount = -1; // -1 only during first iteration
        int nextPollTimeoutMillis = 0;
        for (;;) {
            if (nextPollTimeoutMillis != 0) {
                Binder.flushPendingCommands();
            }

// 看这里看这里看这里
            nativePollOnce(ptr, nextPollTimeoutMillis);

            synchronized (this) {
                // Try to retrieve the next message.  Return if found.
                final long now = SystemClock.uptimeMillis();
                Message prevMsg = null;
                Message msg = mMessages;
                if (msg != null && msg.target == null) {
                    // Stalled by a barrier.  Find the next asynchronous message in the queue.
                    do {
                        prevMsg = msg;
                        msg = msg.next;
                    } while (msg != null && !msg.isAsynchronous());
                }
                if (msg != null) {
                    if (now < msg.when) {
                        // Next message is not ready.  Set a timeout to wake up when it is ready.
                        nextPollTimeoutMillis = (int) Math.min(msg.when - now, Integer.MAX_VALUE);
                    } else {
                        // Got a message.
                        mBlocked = false;
                        if (prevMsg != null) {
                            prevMsg.next = msg.next;
                        } else {
                            mMessages = msg.next;
                        }
                        msg.next = null;
                        if (DEBUG) Log.v(TAG, "Returning message: " + msg);
                        msg.markInUse();
                        return msg;
                    }
                } else {
                    // No more messages.
                    nextPollTimeoutMillis = -1;
                }

                // Process the quit message now that all pending messages have been handled.
                if (mQuitting) {
                    dispose();
                    return null;
                }

                // If first time idle, then get the number of idlers to run.
                // Idle handles only run if the queue is empty or if the first message
                // in the queue (possibly a barrier) is due to be handled in the future.
                if (pendingIdleHandlerCount < 0
                        && (mMessages == null || now < mMessages.when)) {
                    pendingIdleHandlerCount = mIdleHandlers.size();
                }
                if (pendingIdleHandlerCount <= 0) {
                    // No idle handlers to run.  Loop and wait some more.
                    mBlocked = true;
                    continue;
                }

                if (mPendingIdleHandlers == null) {
                    mPendingIdleHandlers = new IdleHandler[Math.max(pendingIdleHandlerCount, 4)];
                }
                mPendingIdleHandlers = mIdleHandlers.toArray(mPendingIdleHandlers);
            }

            // Run the idle handlers.
            // We only ever reach this code block during the first iteration.
            for (int i = 0; i < pendingIdleHandlerCount; i++) {
                final IdleHandler idler = mPendingIdleHandlers[i];
                mPendingIdleHandlers[i] = null; // release the reference to the handler

                boolean keep = false;
                try {
                    keep = idler.queueIdle();
                } catch (Throwable t) {
                    Log.wtf(TAG, "IdleHandler threw exception", t);
                }

                if (!keep) {
                    synchronized (this) {
                        mIdleHandlers.remove(idler);
                    }
                }
            }

            // Reset the idle handler count to 0 so we do not run them again.
            pendingIdleHandlerCount = 0;

            // While calling an idle handler, a new message could have been delivered
            // so go back and look again for a pending message without waiting.
            nextPollTimeoutMillis = 0;
        }
    }
```
这里的大概流程就是按照当前的队列找到 队头的message 取出并返回，然后继续等待。
如果有下一个message则等待 一段时间（视下一个message的delay决定）或者等待唤醒。

还差最后一步2,那么是谁来嗲用message 的next的呢？
是Looper
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

        // Allow overriding a threshold with a system prop. e.g.
        // adb shell 'setprop log.looper.1000.main.slow 1 && stop && start'
        final int thresholdOverride =
                SystemProperties.getInt("log.looper."
                        + Process.myUid() + "."
                        + Thread.currentThread().getName()
                        + ".slow", 0);

        boolean slowDeliveryDetected = false;

        for (;;) {
            Message msg = queue.next(); // might block
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
            long slowDispatchThresholdMs = me.mSlowDispatchThresholdMs;
            long slowDeliveryThresholdMs = me.mSlowDeliveryThresholdMs;
            if (thresholdOverride > 0) {
                slowDispatchThresholdMs = thresholdOverride;
                slowDeliveryThresholdMs = thresholdOverride;
            }
            final boolean logSlowDelivery = (slowDeliveryThresholdMs > 0) && (msg.when > 0);
            final boolean logSlowDispatch = (slowDispatchThresholdMs > 0);

            final boolean needStartTime = logSlowDelivery || logSlowDispatch;
            final boolean needEndTime = logSlowDispatch;

            if (traceTag != 0 && Trace.isTagEnabled(traceTag)) {
                Trace.traceBegin(traceTag, msg.target.getTraceName(msg));
            }

            final long dispatchStart = needStartTime ? SystemClock.uptimeMillis() : 0;
            final long dispatchEnd;
            try {
                msg.target.dispatchMessage(msg);
                dispatchEnd = needEndTime ? SystemClock.uptimeMillis() : 0;
            } finally {
                if (traceTag != 0) {
                    Trace.traceEnd(traceTag);
                }
            }
            if (logSlowDelivery) {
                if (slowDeliveryDetected) {
                    if ((dispatchStart - msg.when) <= 10) {
                        Slog.w(TAG, "Drained");
                        slowDeliveryDetected = false;
                    }
                } else {
                    if (showSlowLog(slowDeliveryThresholdMs, msg.when, dispatchStart, "delivery",
                            msg)) {
                        // Once we write a slow delivery log, suppress until the queue drains.
                        slowDeliveryDetected = true;
                    }
                }
            }
            if (logSlowDispatch) {
                showSlowLog(slowDispatchThresholdMs, dispatchStart, dispatchEnd, "dispatch", msg);
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

            msg.recycleUnchecked();
        }
    }
```
Looper.loop() 方法是一个无限循环，从messagequeue 取出消息之后就dispatch分发这个message处理`msg.target.dispatchMessage(msg);`
那么现在我们可以看到这handler的大郅流程如下![未命名文件 (1).png](https://upload-images.jianshu.io/upload_images/12975041-f300b22d5a958a12.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

looper 从mq 中去取数据 如果取不到就等待block在next上。
取到之后就在loop() 方法里面就地分发。
而handler则负责往mq 里面塞message。

至于最开始
```java
Handler handler = new Handler() {
            @Override
            public void handleMessage(Message msg) {
                super.handleMessage(msg);
                switch (msg.what) {
                    case 1:
                        break;
                }
            }
        };
```
里面的handlemessage 怎么回调的，那我们再从头追述一下。
```java
//Handler.java
private boolean enqueueMessage(MessageQueue queue, Message msg, long uptimeMillis) {
        msg.target = this;//看这里
        if (mAsynchronous) {
            msg.setAsynchronous(true);
        }
        return queue.enqueueMessage(msg, uptimeMillis);
    }

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
```
看到没有这里有个msg.target = this;而这一句也是我们常说的 handler容易导致内存泄漏的问题。

```java
// Looper.java
msg.target.dispatchMessage(msg);
```
那么结合刚刚看的looper的源码就可以知道 Looper 中调用了 handler的 dispatchMessage
而dispatchMessage 中就调用了hanlderMessage
从这个方法里面我们也可以看到  如果设置了 callback 就不会调用 handleMessage了。


结合上一篇Handler 整体流程通关 的源码初步分析，我们一个一个来分析问题：
#一个线程可以有几个handler
如果使用过handler的同学都知道在实际使用过程中可以有无数个handler。

#一个线程可以有几个looper
一个线程可以有几个looper 那么就要分析looper的源码
```java

    private static void prepare(boolean quitAllowed) {
        if (sThreadLocal.get() != null) {
            throw new RuntimeException("Only one Looper may be created per thread");
        }
        sThreadLocal.set(new Looper(quitAllowed));
    }

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
Looper在使用之前必须prepare 在prepare 里面使用了 ThreadLocal，[看我的这篇文章](https://www.jianshu.com/p/d4b2c9ce0aad)
在这里保证了一个线程只能有一个Looper

#handler的内存泄漏的原因？为什么其他的内部类没有提说过有这个问题？
在上一篇[源码](https://www.jianshu.com/p/9ab4163337a6)分析的文章里面已经说过了啊，在调用到 `enqueueMessage `的是否会将 handler赋值给 message的target，而Message 在 messagequeue中等待的时候如果生命周期发生变化但是message中的target不释放就会导致泄漏，毕竟**内存泄漏就是生命周期不相等导致的**

#为何主线程可以new Handler？ 如果想要在子线程中new Handler要做些什么准备？
```java
        new Thread() {
            @Override
            public void run() {
                Handler handler = new Handler();
                handler.post(new Runnable() {
                    @Override
                    public void run() {

                    }
                });
            }
        }.start();
```
我们在子线程中这样运行是不会生效的
```java
    public boolean sendMessageAtTime(Message msg, long uptimeMillis) {
        MessageQueue queue = mQueue;
        if (queue == null) {
            RuntimeException e = new RuntimeException(
                    this + " sendMessageAtTime() called with no mQueue");
            Log.w("Looper", e.getMessage(), e);
            return false;
        }
        return enqueueMessage(queue, msg, uptimeMillis);
    }
```
所以为什么主线程的Handler 可以正确运行，必然是主线程的Handler里面的MessageQueue 是ready的 而我们又知道 messagequeue来之于looper。所以主线程必然在某个时间准备好了Looper，那么我们就去看主线程的启动地方。
```java
//ActivityThread.java
    public static void main(String[] args) {
        Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "ActivityThreadMain");

        // CloseGuard defaults to true and can be quite spammy.  We
        // disable it here, but selectively enable it later (via
        // StrictMode) on debug builds, but using DropBox, not logs.
        CloseGuard.setEnabled(false);

        Environment.initForCurrentUser();

        // Set the reporter for event logging in libcore
        EventLogger.setReporter(new EventLoggingReporter());

        // Make sure TrustedCertificateStore looks in the right place for CA certificates
        final File configDir = Environment.getUserConfigDirectory(UserHandle.myUserId());
        TrustedCertificateStore.setDefaultUserDirectory(configDir);

        Process.setArgV0("<pre-initialized>");

//看这里
        Looper.prepareMainLooper();

        // Find the value for {@link #PROC_START_SEQ_IDENT} if provided on the command line.
        // It will be in the format "seq=114"
        long startSeq = 0;
        if (args != null) {
            for (int i = args.length - 1; i >= 0; --i) {
                if (args[i] != null && args[i].startsWith(PROC_START_SEQ_IDENT)) {
                    startSeq = Long.parseLong(
                            args[i].substring(PROC_START_SEQ_IDENT.length()));
                }
            }
        }
        ActivityThread thread = new ActivityThread();
        thread.attach(false, startSeq);

        if (sMainThreadHandler == null) {
            sMainThreadHandler = thread.getHandler();
        }

        if (false) {
            Looper.myLooper().setMessageLogging(new
                    LogPrinter(Log.DEBUG, "ActivityThread"));
        }

        // End of event ActivityThreadMain.
        Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
        Looper.loop();

        throw new RuntimeException("Main thread loop unexpectedly exited");
    }
```
在这个方法里面，looper在`Looper.prepareMainLooper();` 之后做好准备工作之后 就开始调用loop 开始取消息，所有的 生命周期，我们都是通过message来驱动的。Looper中`private static Looper sMainLooper;` 存在这个变量更是确保了一个进程只有一个 sMainLooper，当然会有一些锁来确保线程安全。
**如果想要在子线程中new Handler要做些什么准备？**
在创建之前 调用looper的prepare 即可。
```java
 new Thread() {
            @Override
            public void run() {
                Looper.prepare();
                Handler handler = new Handler();
                handler.post(new Runnable() {
                    @Override
                    public void run() {

                    }
                });
                Looper.loop();
            }
        }.start();
```

#子线程中维护的Looper，消息队列无消息的时候的处理方案是什么？有什么用？
这里有两种解释。
- 第一种：上一篇文章我们就说到了 loop() 方法会不断的去调用messagequeue的next() 方法。代码就不贴了，最终会在调用`nativePollOnce(ptr, nextPollTimeoutMillis);` 这一步时卡住，此时线程就休眠了，等待下一次的`nativeWake` 或者timeout自动唤醒。
这样做有什么好处呢，节约cpu。
- 第二种：当消息队列无消息的时候以为着什么 这个子线程已经无用了，可以推出了，这时候问的就是该如何退出。
```java
//Looper.java
    public void quit() {
        mQueue.quit(false);
    }
    public void quitSafely() {
        mQueue.quit(true);
    }
```
```java
//MessageQueue.java
    void quit(boolean safe) {
        if (!mQuitAllowed) {
            throw new IllegalStateException("Main thread not allowed to quit.");
        }

        synchronized (this) {
            if (mQuitting) {
                return;
            }
            mQuitting = true;

            if (safe) {
                removeAllFutureMessagesLocked();
            } else {
                removeAllMessagesLocked();
            }

            // We can assume mPtr != 0 because mQuitting was previously false.
            nativeWake(mPtr);
        }
    }

    private void removeAllMessagesLocked() {
        Message p = mMessages;
        while (p != null) {
            Message n = p.next;
            p.recycleUnchecked();
            p = n;
        }
        mMessages = null;
    }

    private void removeAllFutureMessagesLocked() {
        final long now = SystemClock.uptimeMillis();
        Message p = mMessages;
        if (p != null) {
            if (p.when > now) {
                removeAllMessagesLocked();
            } else {
                Message n;
                for (;;) {
                    n = p.next;
                    if (n == null) {
                        return;
                    }
                    if (n.when > now) {
                        break;
                    }
                    p = n;
                }
                p.next = null;
                do {
                    p = n;
                    n = p.next;
                    p.recycleUnchecked();
                } while (n != null);
            }
        }
    }
```
[quit 和quit safely的区别看这里--未发布]()
这里主要是把message 全部删除。
在Looper.loop 方法中 
```
     for (;;) {
            Message msg = queue.next(); // might block
// 这里如果不退出这里必然返回的不为空，要么block在这，要么返回一个非空值，
//只有在退出时，清空队列，并且唤醒block 这里才会返回null 从而退出
            if (msg == null) {
                // No message indicates that the message queue is quitting.
                return;
            }
。。。
```
当取出的message 为null时将会推出loop ，线程也自动结束了

**有什么用？**
```java
//Message.java
    void recycleUnchecked() {
        // Mark the message as in use while it remains in the recycled object pool.
        // Clear out all other details.
        flags = FLAG_IN_USE;
        what = 0;
        arg1 = 0;
        arg2 = 0;
        obj = null;
        replyTo = null;
        sendingUid = -1;
        when = 0;
        target = null;
        callback = null;
        data = null;

        synchronized (sPoolSync) {
            if (sPoolSize < MAX_POOL_SIZE) {
                next = sPool;
                sPool = this;
                sPoolSize++;
            }
        }
    }
```
在这里会清空message的target callback data等内容，可以释放内存，避免内存泄漏。
#既然可以存在多个handler 往mq中add 数据，那内部是如何确保线程数据安全的？
```java
//MessageQueue.java
    boolean enqueueMessage(Message msg, long when) {
        if (msg.target == null) {
            throw new IllegalArgumentException("Message must have a target.");
        }
        if (msg.isInUse()) {
            throw new IllegalStateException(msg + " This message is already in use.");
        }

        synchronized (this) {
            if (mQuitting) {
                IllegalStateException e = new IllegalStateException(
                        msg.target + " sending message to a Handler on a dead thread");
                Log.w(TAG, e.getMessage(), e);
                msg.recycle();
                return false;
            }
...
```
在这里加锁了，`synchronized (this)` [那么synchronized 锁的内容不同有什么区别呢-- 未发布]()
#我们使用Message 时应该如何创建它？
Message 我们可以直接new 也可以 通过obtain 来获取复用的 message 实例。
那么这里问的就是obtain 的运作逻辑和为什么要用obtain
```java
//Message.java
public static Message obtain() {
        synchronized (sPoolSync) {
            if (sPool != null) {
                Message m = sPool;
                sPool = m.next;
                m.next = null;
                m.flags = 0; // clear in-use flag
                sPoolSize--;
                return m;
            }
        }
        return new Message();
    }
```
这里用了一个享元模式，Message 是Android系统中很常见的一个类，如果我们每次使用都new的话 ，这个内存就抖动起来了呀。 这里也是为了避免过多的内存创建销毁的过程，平稳使用内存。
#使用Handler的postDelay后消息队列会有什么变化
```java
//Handler.java
   public final boolean postDelayed(Runnable r, long delayMillis)
    {
        return sendMessageDelayed(getPostMessage(r), delayMillis);
    }
    public final boolean sendMessageDelayed(Message msg, long delayMillis)
    {
        if (delayMillis < 0) {
            delayMillis = 0;
        }
        return sendMessageAtTime(msg, SystemClock.uptimeMillis() + delayMillis);
    }
    public boolean sendMessageAtTime(Message msg, long uptimeMillis) {
        MessageQueue queue = mQueue;
        if (queue == null) {
            RuntimeException e = new RuntimeException(
                    this + " sendMessageAtTime() called with no mQueue");
            Log.w("Looper", e.getMessage(), e);
            return false;
        }
        return enqueueMessage(queue, msg, uptimeMillis);
    }
```
我们可以看到在调用enqueueMessage的之前都是传入了一个时间值，而在messagequeue的源码中，message也是按照时间顺序排列的 一个队列。在插入之后会查看是否需要wake。如果我们插入的是一个time为0的时候messagequeue会立刻唤醒，如果插入的是一个延迟任务，将不会唤醒messagequeue。

# looper死循环为什么不会导致应用卡死？
- [源码](http://androidxref.com/8.1.0_r33/xref/frameworks/base/services/core/java/com/android/server/am/ActivityManagerService.java)
- 借用这篇文章你可以知道service boradcast provider [ anr]( http://gityuan.com/2016/07/02/android-anr/) 的原理
- 借用这篇文章 你可以知道 input的 [anr](http://gityuan.com/2017/01/01/input-anr/)

这里你可以看到 为什么会anr 总结起来都是 有某些地方发送了 timeout的 message ，然后hander 处理了这个message。
而当 looper 就是循环读取messagequeue的message 然后处理，如果没有message 就等待 这时候 的等待是不可能达成这个anr的逻辑的。

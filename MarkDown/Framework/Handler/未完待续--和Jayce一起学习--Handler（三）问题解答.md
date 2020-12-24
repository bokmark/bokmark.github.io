> [Jayce 带你学习Handler（一）](https://www.jianshu.com/p/aa17a07aaae7)
> [Jayce 带你学习Handler（二）Handler 整体流程通关](https://www.jianshu.com/p/9ab4163337a6)


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

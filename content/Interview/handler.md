---
author: Bokmark Ma
title: "各种面试的答案 - Handler"
date: 2022-10-16T00:35:27+08:00
description: 面试题 结合
slug: interview/handler
categories:
    - Interview
tags:
    - 面试
    - Handler
---

# 各种面试的答案 - Handler
 

## Handler原理，post 和 send的区别
> Handler是一个死循环处理消息的工具。创建Hanlder的时候必须传入一个Looper，否则就获取当前线程的looper。looper中含有一个messagequeue，当开发人员调用hanlder的post或者send方法时，将会在looper的messagequeue中插入一个message。而looper将会阻塞在messagequeue的next方法中，一旦messagequeue中有消息时，next方法将会唤醒，然后从messagequeue中依次拿出message处理。  
post和send的区别在于，post传入的参数是runnable，handler将会把这个runnable打包成一个message，然后调用send方法插入looper的message queue。而send方法传入的就是message则是直接插入mq。  
其中有几个需要关注的重点，looper通过threadlocal来获取当前线程的looper，主线程调用的是preparemainlooper（ActivityThread类中），messageQueue以时间为权重将message放到队列中，next方法采用的是linux的epoll机制并不会产生ANR（todo why），message的执行将会由产生它的handler来处理（message的target引用来handler，这可能引起内存泄露）

那么看下[源码](http://androidxref.com/9.0.0_r3/xref/frameworks/base/core/java/android/os/Handler.java)：
```java
// 
public class Handler {
     public Handler() {
        this(null, false);
    }
    public Handler(Looper looper) {
        this(looper, null, false);
    }
    public Handler(Looper looper, Callback callback) {
        this(looper, callback, false);
    }
    public Handler(Looper looper, Callback callback, boolean async) {
        mLooper = looper;
        mQueue = looper.mQueue;
        mCallback = callback;
        mAsynchronous = async;
    }

    final Looper mLooper;
    final MessageQueue mQueue;
    final Callback mCallback;
    final boolean mAsynchronous;


    public boolean sendMessageAtTime(Message msg, long uptimeMillis) {
        MessageQueue queue = mQueue;
        if (queue == null) {
            RuntimeException e = new RuntimeException(this + " sendMessageAtTime() called with no mQueue");
            return false;
        }
        return enqueueMessage(queue, msg, uptimeMillis);
    }
    
    private boolean enqueueMessage(MessageQueue queue, Message msg, long uptimeMillis) {
        msg.target = this;
        if (mAsynchronous) {
            msg.setAsynchronous(true);
        }
        return queue.enqueueMessage(msg, uptimeMillis);
    }

    public final boolean postAtFrontOfQueue(Runnable r) {
        return sendMessageAtFrontOfQueue(getPostMessage(r));
    }

    private static Message getPostMessage(Runnable r) {
        Message m = Message.obtain(); //【bokmark】：全局的message池
        m.callback = r;
        return m;
    }
}
```
首先从这个`Handler.java`可以看出send和post之间的区别，send是直接往messagequeue里面插入一个message，post则是将runnable打包成一个message，然后在通过send方法插入messagequeue。  
而looper是如何死循环取出message的呢？看[源码](http://androidxref.com/9.0.0_r3/xref/frameworks/base/core/java/android/os/Looper.java)：
```java
public final class Looper {
    // sThreadLocal.get() will return null unless you've called prepare().
    static final ThreadLocal<Looper> sThreadLocal = new ThreadLocal<Looper>();
    private static Looper sMainLooper;  // guarded by Looper.class

    private Looper(boolean quitAllowed) {
        mQueue = new MessageQueue(quitAllowed);
        mThread = Thread.currentThread();
    }

    private static void prepare(boolean quitAllowed) {
        if (sThreadLocal.get() != null) {
            throw new RuntimeException("Only one Looper may be created per thread");
        }
        sThreadLocal.set(new Looper(quitAllowed));
    }
    // 
    public static void prepareMainLooper() {
        prepare(false);
        synchronized (Looper.class) {
            if (sMainLooper != null) {
                throw new IllegalStateException("The main Looper has already been prepared.");
            }
            sMainLooper = myLooper();
        }
    }
    public static Looper getMainLooper() {
        synchronized (Looper.class) {
            return sMainLooper;
        }
    }
    public static void loop() {
        final Looper me = myLooper();
        if (me == null) {
            throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
        }
        final MessageQueue queue = me.mQueue;
        // .....
        for (;;) {
            Message msg = queue.next(); // might block
            if (msg == null) {
                // No message indicates that the message queue is quitting.
                return;
            }
            // ............
            try {
                //【bokmark】：target 就是 插入 message对应的handler。
                msg.target.dispatchMessage(msg);
                // .....
            }
            // .... 
            msg.recycleUnchecked();
        }
    }
}
```
从`Looper.java`中可以看到messageQueue的next方法将会发生阻塞等待下一个可执行的message的到来。而ThreadLocal用于将looper保存到线程中。
而messagequeue为什么会阻塞，而且不产生ANR? 先看[源码](http://androidxref.com/9.0.0_r3/xref/frameworks/base/core/java/android/os/MessageQueue.java)：
```java
public final class MessageQueue {

    Message next() {
        // ....
        for (;;) {
            // ....
            nativePollOnce(ptr, nextPollTimeoutMillis);

            synchronized (this) {
                // Try to retrieve the next message.  Return if found.
                final long now = SystemClock.uptimeMillis();
                Message prevMsg = null;
                Message msg = mMessages;
                if (msg != null && msg.target == null) {
                    // Stalled by a barrier.  Find the next asynchronous message in the queue.
                    // 同步屏障，后文会提到
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
                // ....
            }
        }
    }
}
```
从这里的next方法可以看出处理逻辑。  
1. 判断target是否为空，如果为空则找到异步message，抛弃之前的所有message。（同步屏障）否则，
2. 如果message的执行时间还未到，则陷入poll。否则，
3. 取出message，返回。  

那么 `nativePollOnce` 是怎么实现的呢？看[代码](http://androidxref.com/9.0.0_r3/xref/frameworks/base/core/jni/android_os_MessageQueue.cpp)：
```c++

// jni 本地方法和java方法的注册表
static const JNINativeMethod gMessageQueueMethods[] = {
    /* name, signature, funcPtr */
    { "nativeInit", "()J", (void*)android_os_MessageQueue_nativeInit },
    { "nativeDestroy", "(J)V", (void*)android_os_MessageQueue_nativeDestroy },
    { "nativePollOnce", "(JI)V", (void*)android_os_MessageQueue_nativePollOnce },
    { "nativeWake", "(J)V", (void*)android_os_MessageQueue_nativeWake },
    { "nativeIsPolling", "(J)Z", (void*)android_os_MessageQueue_nativeIsPolling },
    { "nativeSetFileDescriptorEvents", "(JII)V",
            (void*)android_os_MessageQueue_nativeSetFileDescriptorEvents },
};
// 调用Java的nativePollOnce将会走到这个方法
static void android_os_MessageQueue_nativePollOnce(JNIEnv* env, jobject obj,
        jlong ptr, jint timeoutMillis) {
    NativeMessageQueue* nativeMessageQueue = reinterpret_cast<NativeMessageQueue*>(ptr);
    nativeMessageQueue->pollOnce(env, obj, timeoutMillis);
}
// 具体的实现
void NativeMessageQueue::pollOnce(JNIEnv* env, jobject pollObj, int timeoutMillis) {
    mPollEnv = env;
    mPollObj = pollObj;
    mLooper->pollOnce(timeoutMillis);
    mPollObj = NULL;
    mPollEnv = NULL;

    if (mExceptionObj) {
        env->Throw(mExceptionObj);
        env->DeleteLocalRef(mExceptionObj);
        mExceptionObj = NULL;
    }
}

```
忽略到中间各种调用，具体来到[Looper.cpp](http://androidxref.com/9.0.0_r3/xref/system/core/libutils/Looper.cpp)
```c++
Looper::Looper(bool allowNonCallbacks) : xxx {
    mWakeEventFd = eventfd(0, EFD_NONBLOCK | EFD_CLOEXEC);
    //...
    AutoMutex _l(mLock);
    rebuildEpollLocked();
}

void Looper::rebuildEpollLocked() { 
    // 创建一个Epoll
    mEpollFd = epoll_create(EPOLL_SIZE_HINT);
    LOG_ALWAYS_FATAL_IF(mEpollFd < 0, "Could not create epoll instance: %s", strerror(errno));

    struct epoll_event eventItem;
    // 一系列初始化
    for (size_t i = 0; i < mRequests.size(); i++) { 
        // 进入内核态，添加一个epoll
        int epollResult = epoll_ctl(mEpollFd, EPOLL_CTL_ADD, request.fd, & eventItem);
        if (epollResult < 0) {
            ALOGE("Error adding epoll events for fd %d while rebuilding epoll set: %s",
                  request.fd, strerror(errno));
        }
    }
}
int Looper::pollOnce(int timeoutMillis, int* outFd, int* outEvents, void** outData) {
    int result = 0;
    for (;;) {
        // 检查工作
        result = pollInner(timeoutMillis);
    }
}

int Looper::pollInner(int timeoutMillis) { 
    // ....
    // 等待唤醒
    int eventCount = epoll_wait(mEpollFd, eventItems, EPOLL_MAX_EVENTS, timeoutMillis);
    // 处理结果
}
```
重点还在于`epoll_create`和`epoll_wait` 这是linux的epoll机制，可以看这篇文章理解[epoll](https://zhuanlan.zhihu.com/p/64746509)
简单理解，当进入Wait阶段时，CPU将当前进程加入一个等待队列，一旦message就绪，将会触发中断程序，CPU将当前进程唤醒进程处理，这样就可以避免CPU的空耗。

## idle
当hanlder空闲时，可以添加一些任务交由hanlder来处理。那么是如何实现的呢？还得看[源码](http://androidxref.com/9.0.0_r3/xref/frameworks/base/core/java/android/os/MessageQueue.java#IdleHandler):
```java
public final class MessageQueue {

    private final ArrayList<IdleHandler> mIdleHandlers = new ArrayList<IdleHandler>();
    private IdleHandler[] mPendingIdleHandlers;
    
    public void addIdleHandler(@NonNull IdleHandler handler) {
        if (handler == null) {
            throw new NullPointerException("Can't add a null IdleHandler");
        }
        synchronized (this) {
            mIdleHandlers.add(handler);
        }
    }

    public void removeIdleHandler(@NonNull IdleHandler handler) {
        synchronized (this) {
            mIdleHandlers.remove(handler);
        }
    }
    Message next() {
        int pendingIdleHandlerCount = -1;
        for (;;) {
            //....
            if (pendingIdleHandlerCount < 0 && (mMessages == null || now < mMessages.when)) {
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
 
            pendingIdleHandlerCount = 0;
            //.....
        }  
    public static interface IdleHandler {
        /**
            * Called when the message queue has run out of messages and will now
            * wait for more.  Return true to keep your idle handler active, false
            * to have it removed.  This may be called if there are still messages
            * pending in the queue, but they are all scheduled to be dispatched
            * after the current time.
            */
        boolean queueIdle();
    }
}
```
- 当looper不断的取数据，从而不断调用messagequeue的next方法时，会判断mIdleHandlers是否有未处理的任务
- 如果没有，则等待下一个消息的到来；如果有，则将所有的mIdleHandlers遍历一遍，将返回false的移除。
- 如果返回true，在下次空闲的时候，该任务将被再次执行。

## 同步屏障在什么时候可能用到
在前文MessageQueue的next方法中有这么一段
```java
public final class MessageQueue {

    Message next() {
        // ....
        for (;;) {
            // .... 
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
                // ....
            }
        }
    }
}
```
这里的代码如果碰到某个messag的target为空，就会去寻找一个异步的message并且丢弃这个异步方法之前的所有message。  
target在post或者send的时候都会被handler设置为this，那么什么情况下会被设置为空呢，还有什么情况下message isAsynchronous 会返回false，什么时候返回true呢？  
先看第一个，什么时候target为空：
```java
public final class MessageQueue {
    /**
     * Posts a synchronization barrier to the Looper's message queue.
     *
     * Message processing occurs as usual until the message queue encounters the
     * synchronization barrier that has been posted.  When the barrier is encountered,
     * later synchronous messages in the queue are stalled (prevented from being executed)
     * until the barrier is released by calling {@link #removeSyncBarrier} and specifying
     * the token that identifies the synchronization barrier.
     *
     * This method is used to immediately postpone execution of all subsequently posted
     * synchronous messages until a condition is met that releases the barrier.
     * Asynchronous messages (see {@link Message#isAsynchronous} are exempt from the barrier
     * and continue to be processed as usual.
     *
     * This call must be always matched by a call to {@link #removeSyncBarrier} with
     * the same token to ensure that the message queue resumes normal operation.
     * Otherwise the application will probably hang!
     *
     * @return A token that uniquely identifies the barrier.  This token must be
     * passed to {@link #removeSyncBarrier} to release the barrier.
     *
     * @hide
     */
    public int postSyncBarrier() {
        return postSyncBarrier(SystemClock.uptimeMillis());
    }  
    
    private int postSyncBarrier(long when) {
        // Enqueue a new sync barrier token.
        // We don't need to wake the queue because the purpose of a barrier is to stall it.
        synchronized (this) {
            final int token = mNextBarrierToken++;
            final Message msg = Message.obtain();
            msg.markInUse();
            msg.when = when;
            msg.arg1 = token;

            Message prev = null;
            Message p = mMessages;
            if (when != 0) {
                while (p != null && p.when <= when) {
                    prev = p;
                    p = p.next;
                }
            }
            if (prev != null) { // invariant: p == prev.next
                msg.next = p;
                prev.next = msg;
            } else {
                msg.next = p;
                mMessages = msg;
            }
            return token;
        }
    }

}
```
而postSyncBarrier 什么时候被调用呢? [ViewRootImpl.java](http://androidxref.com/9.0.0_r3/xref/frameworks/base/core/java/android/view/ViewRootImpl.java)
```java
void scheduleTraversals() {
    if (!mTraversalScheduled) {
        mTraversalScheduled = true;
        mTraversalBarrier = mHandler.getLooper().getQueue().postSyncBarrier();
        mChoreographer.postCallback(
                Choreographer.CALLBACK_TRAVERSAL, mTraversalRunnable, null);
        if (!mUnbufferedInputDispatch) {
            scheduleConsumeBatchedInput();
        }
        notifyRendererOfFramePending();
        pokeDrawLockIfNeeded();
    }
}
```
scheduleTraversals方法是在当ui可能需要进行重新绘制时调用，而当ui重新绘制时，那么主线程就需要迅速响应绘制，避免ui卡顿。  
那么什么时候，message变为同步message来呢。我们搜索`setAsynchronous`方法，[结果](http://androidxref.com/9.0.0_r3/s?refs=setAsynchronous&project=frameworks)有很多地方，粗看几个，你就会发现都是一些ui交互等很重要的动作，比如有用户输入，系统的电量信息更新等等。  

---
title: 'OkHTTP 整体流程'
date: 2020-12-01T15:14:39+10:00
weight: 1
tags: ["tcp、ip"]
categories: ["网络协议"]
---

# okhttp
https://blog.csdn.net/chunqiuwei/article/details/71939952

[github 源地址](https://github.com/square/okhttp)

## 整体流程
{{<mermaid>}}
sequenceDiagram
    participant c as OkHttpClient
    participant rc as RealCall
    participant d as Dispatchers
    participant chain as RealInterceptorChain
    c->>rc: RealCall.newRealCall(this, request, false)
    rc-->>c: new RealCall(client, originalRequest, forWebSocket);
    rc->>d: enqueue()/*见详细解释*/
    d-->rc: executeOn()/*见详细解释*/
    rc-->>rc: execute()
    rc->>chain: getResponseWithInterceptorChain()
    chain-->>c: callback

{{</mermaid>}}


## 分发器
{{<details "Dispatcher.java">}}

```Java

public final class Dispatcher {
  private int maxRequests = 64;
  private int maxRequestsPerHost = 5;
  private @Nullable Runnable idleCallback;

  /** Executes calls. Created lazily. */
  private @Nullable ExecutorService executorService;

  /** Ready async calls in the order they'll be run. */
  private final Deque<AsyncCall> readyAsyncCalls = new ArrayDeque<>();

  /** Running asynchronous calls. Includes canceled calls that haven't finished yet. */
  private final Deque<AsyncCall> runningAsyncCalls = new ArrayDeque<>();

  /** Running synchronous calls. Includes canceled calls that haven't finished yet. */
  private final Deque<RealCall> runningSyncCalls = new ArrayDeque<>();

  public Dispatcher(ExecutorService executorService) {
    this.executorService = executorService;
  }

  public Dispatcher() {
  }

  public synchronized ExecutorService executorService() {
    if (executorService == null) {
      executorService = new ThreadPoolExecutor(0, Integer.MAX_VALUE, 60, TimeUnit.SECONDS,
          new SynchronousQueue<>(), Util.threadFactory("OkHttp Dispatcher", false));
    }
    return executorService;
  }

  /**
   * Set the maximum number of requests to execute concurrently. Above this requests queue in
   * memory, waiting for the running calls to complete.
   *
   * <p>If more than {@code maxRequests} requests are in flight when this is invoked, those requests
   * will remain in flight.
   */
  public void setMaxRequests(int maxRequests) {
    if (maxRequests < 1) {
      throw new IllegalArgumentException("max < 1: " + maxRequests);
    }
    synchronized (this) {
      this.maxRequests = maxRequests;
    }
    promoteAndExecute();
  }

  public synchronized int getMaxRequests() {
    return maxRequests;
  }

  /**
   * Set the maximum number of requests for each host to execute concurrently. This limits requests
   * by the URL's host name. Note that concurrent requests to a single IP address may still exceed
   * this limit: multiple hostnames may share an IP address or be routed through the same HTTP
   * proxy.
   *
   * <p>If more than {@code maxRequestsPerHost} requests are in flight when this is invoked, those
   * requests will remain in flight.
   *
   * <p>WebSocket connections to hosts <b>do not</b> count against this limit.
   */
  public void setMaxRequestsPerHost(int maxRequestsPerHost) {
    if (maxRequestsPerHost < 1) {
      throw new IllegalArgumentException("max < 1: " + maxRequestsPerHost);
    }
    synchronized (this) {
      this.maxRequestsPerHost = maxRequestsPerHost;
    }
    promoteAndExecute();
  }

  public synchronized int getMaxRequestsPerHost() {
    return maxRequestsPerHost;
  }

  /**
   * Set a callback to be invoked each time the dispatcher becomes idle (when the number of running
   * calls returns to zero).
   *
   * <p>Note: The time at which a {@linkplain Call call} is considered idle is different depending
   * on whether it was run {@linkplain Call#enqueue(Callback) asynchronously} or
   * {@linkplain Call#execute() synchronously}. Asynchronous calls become idle after the
   * {@link Callback#onResponse onResponse} or {@link Callback#onFailure onFailure} callback has
   * returned. Synchronous calls become idle once {@link Call#execute() execute()} returns. This
   * means that if you are doing synchronous calls the network layer will not truly be idle until
   * every returned {@link Response} has been closed.
   */
  public synchronized void setIdleCallback(@Nullable Runnable idleCallback) {
    this.idleCallback = idleCallback;
  }

  void enqueue(AsyncCall call) {
    synchronized (this) {
      readyAsyncCalls.add(call);

      // Mutate the AsyncCall so that it shares the AtomicInteger of an existing running call to
      // the same host.
      if (!call.get().forWebSocket) {
        AsyncCall existingCall = findExistingCallWithHost(call.host());
        if (existingCall != null) call.reuseCallsPerHostFrom(existingCall);
      }
    }
    promoteAndExecute();
  }

  @Nullable private AsyncCall findExistingCallWithHost(String host) {
    for (AsyncCall existingCall : runningAsyncCalls) {
      if (existingCall.host().equals(host)) return existingCall;
    }
    for (AsyncCall existingCall : readyAsyncCalls) {
      if (existingCall.host().equals(host)) return existingCall;
    }
    return null;
  }

  /**
   * Cancel all calls currently enqueued or executing. Includes calls executed both {@linkplain
   * Call#execute() synchronously} and {@linkplain Call#enqueue asynchronously}.
   */
  public synchronized void cancelAll() {
    for (AsyncCall call : readyAsyncCalls) {
      call.get().cancel();
    }

    for (AsyncCall call : runningAsyncCalls) {
      call.get().cancel();
    }

    for (RealCall call : runningSyncCalls) {
      call.cancel();
    }
  }

  /**
   * Promotes eligible calls from {@link #readyAsyncCalls} to {@link #runningAsyncCalls} and runs
   * them on the executor service. Must not be called with synchronization because executing calls
   * can call into user code.
   *
   * @return true if the dispatcher is currently running calls.
   */
  private boolean promoteAndExecute() {
    assert (!Thread.holdsLock(this));

    List<AsyncCall> executableCalls = new ArrayList<>();
    boolean isRunning;
    synchronized (this) {
      for (Iterator<AsyncCall> i = readyAsyncCalls.iterator(); i.hasNext(); ) {
        AsyncCall asyncCall = i.next();

        if (runningAsyncCalls.size() >= maxRequests) break; // Max capacity.
        if (asyncCall.callsPerHost().get() >= maxRequestsPerHost) continue; // Host max capacity.

        i.remove();
        asyncCall.callsPerHost().incrementAndGet();
        executableCalls.add(asyncCall);
        runningAsyncCalls.add(asyncCall);
      }
      isRunning = runningCallsCount() > 0;
    }

    for (int i = 0, size = executableCalls.size(); i < size; i++) {
      AsyncCall asyncCall = executableCalls.get(i);
      asyncCall.executeOn(executorService());
    }

    return isRunning;
  }

  /** Used by {@code Call#execute} to signal it is in-flight. */
  synchronized void executed(RealCall call) {
    runningSyncCalls.add(call);
  }

  /** Used by {@code AsyncCall#run} to signal completion. */
  void finished(AsyncCall call) {
    call.callsPerHost().decrementAndGet();
    finished(runningAsyncCalls, call);
  }

  /** Used by {@code Call#execute} to signal completion. */
  void finished(RealCall call) {
    finished(runningSyncCalls, call);
  }

  private <T> void finished(Deque<T> calls, T call) {
    Runnable idleCallback;
    synchronized (this) {
      if (!calls.remove(call)) throw new AssertionError("Call wasn't in-flight!");
      idleCallback = this.idleCallback;
    }

    boolean isRunning = promoteAndExecute();

    if (!isRunning && idleCallback != null) {
      idleCallback.run();
    }
  }

  /** Returns a snapshot of the calls currently awaiting execution. */
  public synchronized List<Call> queuedCalls() {
    List<Call> result = new ArrayList<>();
    for (AsyncCall asyncCall : readyAsyncCalls) {
      result.add(asyncCall.get());
    }
    return Collections.unmodifiableList(result);
  }

  /** Returns a snapshot of the calls currently being executed. */
  public synchronized List<Call> runningCalls() {
    List<Call> result = new ArrayList<>();
    result.addAll(runningSyncCalls);
    for (AsyncCall asyncCall : runningAsyncCalls) {
      result.add(asyncCall.get());
    }
    return Collections.unmodifiableList(result);
  }

  public synchronized int queuedCallsCount() {
    return readyAsyncCalls.size();
  }

  public synchronized int runningCallsCount() {
    return runningAsyncCalls.size() + runningSyncCalls.size();
  }
}
```
{{</details>}}

其中几处关键:

- 请求并发限制
```java
  private int maxRequests = 64; // 当前最大请求数量
  private int maxRequestsPerHost = 5; // 每个host 最大请求数量
```

- 线程池的配置
```Java
  public synchronized ExecutorService executorService() {
    if (executorService == null) {
      executorService = new ThreadPoolExecutor(0, Integer.MAX_VALUE, 60, TimeUnit.SECONDS,
          new SynchronousQueue<>(), Util.threadFactory("OkHttp Dispatcher", false));
    }
    return executorService;
  }
```

- 三个call的队列，请求并发限制也是根据队列的长度作为依据
```Java

  /** 等待执行的异步队列, */
  private final Deque<AsyncCall> readyAsyncCalls = new ArrayDeque<>();

  /** 正在执行中的队列，其中包括了哪些被取消的没有完成的任务 */
  private final Deque<AsyncCall> runningAsyncCalls = new ArrayDeque<>();

  /** 正在执行中的同步队列，其中包括了哪些被取消的没有完成的任务 */
  private final Deque<RealCall> runningSyncCalls = new ArrayDeque<>();
```

- promoteAndExecute()
```Java
private boolean promoteAndExecute() {
 assert (!Thread.holdsLock(this)); // 在执行这段代码的时候不允许在加锁情况下运行

 List<AsyncCall> executableCalls = new ArrayList<>();
 boolean isRunning;
 synchronized (this) {
   // 从ready 队列中挑出几个放到 executableCalls
   for (Iterator<AsyncCall> i = readyAsyncCalls.iterator(); i.hasNext(); ) {
     AsyncCall asyncCall = i.next();

     // callsPerHost 每一个host 同用一个atomicinteger 实例
     if (runningAsyncCalls.size() >= maxRequests) break; // Max capacity.
     if (asyncCall.callsPerHost().get() >= maxRequestsPerHost) continue; // Host max capacity.

     i.remove();
     asyncCall.callsPerHost().incrementAndGet();
     executableCalls.add(asyncCall);
     // 挑出来的 call 放到运行的队列中
     runningAsyncCalls.add(asyncCall);
   }
   isRunning = runningCallsCount() > 0;
 }

 // 将挑出的 call 放到线程池运行
 for (int i = 0, size = executableCalls.size(); i < size; i++) {
   AsyncCall asyncCall = executableCalls.get(i);
   asyncCall.executeOn(executorService());
 }

 return isRunning;
}
```
- promoteAndExecute() 方法执行的时机
  - public void setMaxRequests(int maxRequests)
  - public void setMaxRequestsPerHost(int maxRequestsPerHost)
  - void enqueue(AsyncCall call)
  - private <T> void finished(Deque<T> calls, T call)
    - RealCall:protected void execute() // AsyncCall执行的时候
    - RealCall:public Response execute() throws IOException

  总结就是当有新任务来的时候 会挑出可执行的任务

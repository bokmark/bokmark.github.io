---
title: 'Kotlin 协程 - 怎么使用'
date: 2020-12-10T13:14:00+08:00
draft: false
weight: 3
summary: "kotlin协程 几问"
---

> 参考文档 https://www.jianshu.com/u/a324daa6fa19

##### 添加依赖

kotlin的标准库中有kotlin的协程基础的库，我们现在只关注如何使用协程，重要的部分都在这个库里。（原理在下一篇文章中）。

 ``` groovy
implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-core:1.4.2'
 ```

*kotlin 的版本必须在1.4.10 之上*

##### 基础使用suspend 函数

```kotlin
suspend fun a()
```

- 挂起函数必须在协程体或者挂起函数中执行
- 每个挂起函数都有一个挂起点和一个resume
- 每一个挂起函数都是一个可以取消的点



##### 基础使用 launch

```kotlin
fun main() {
    GlobalScope.launch {
        delay(500)
        println("in global scope ${Thread.currentThread().name}")
    }
    Thread.sleep(1000)// 由于globalscope的生命周期是跟随主线程的 所以这里不sleep的话会导致协程被取消
    println("thread sleep ${Thread.currentThread().name}")
}
```

*kotlin 在jvm上的实现是依赖于 java的thread的，所以我们可以在协程体中获取到相关的Thread的信息，并且也可以使用Thread的方法*

###### GlobalScope: 一个带有生命周期的线程调度器

- 我们来看看其中的launch 方法

  ```kotlin
  public fun CoroutineScope.launch(
      context: CoroutineContext = EmptyCoroutineContext,
      start: CoroutineStart = CoroutineStart.DEFAULT,
      block: suspend CoroutineScope.() -> Unit
  ): Job
  ```

  - context：协程上下文。类似于一个map的概念，其中包含了协程运行的各个参数。具体概念之后介绍。

  - start：启动模式

    - DEFAULT：允许协程马上开始执行。允许在协程运行之前取消。

    - LAZY：需要调用job的start方法，来允许协程马上开始执行。

      ```kotlin
      suspend fun main() {
          println("start in ${Thread.currentThread().name}")
          val job = GlobalScope.launch(start = CoroutineStart.LAZY) {
              println("in global scope ${Thread.currentThread().name} ==== start")
              delay(500)
              println("in global scope ${Thread.currentThread().name} ==== end")
          }
          println("job is defined in ${Thread.currentThread().name}")
          job.start()
          Thread.sleep(1000) // 由于globalscope的生命周期是跟随主线程的 所以这里不sleep的话会导致协程被取消
          println("end in ${Thread.currentThread().name}")
      }
      /*
      start in main
      job is defined in main
      in global scope DefaultDispatcher-worker-1 ==== start
      in global scope DefaultDispatcher-worker-1 ==== end
      end in main
      */
      ```

    - ATOMIC：与DEFAULT相似，只是协程一定会运行到第一个挂起点（suspend 函数）

      ```kotlin
      @ExperimentalCoroutinesApi
      suspend fun main() {
          println("start in ${Thread.currentThread().name}")
          val job = GlobalScope.launch(start = CoroutineStart.ATOMIC) {
              println("in global scope ${Thread.currentThread().name} ==== start")
              delay(500)
              println("in global scope ${Thread.currentThread().name} ==== end")
          }
          //job.cancelAndJoin()
          println("end in ${Thread.currentThread().name}")
          //Thread.sleep(1000)
      }
      /*
      start in main
      end in main
      in global scope DefaultDispatcher-worker-2 ==== start
      ====== 也有可能是这样 ====== 为什么会这样？todo
      start in main
      end in main
      */
      ```

    - UNDISPATCHED：先不调度执行，等到第一个挂起点再调度。

      ```kotlin
      @ExperimentalCoroutinesApi
      suspend fun main() {
          println("start in ${Thread.currentThread().name}")
          val job = GlobalScope.launch(start = CoroutineStart.UNDISPATCHED) {
              println("in global scope ${Thread.currentThread().name} ==== start")
              delay(500)
              println("in global scope ${Thread.currentThread().name} ==== end")
          }
          Thread.sleep(1000)
          println("end in ${Thread.currentThread().name}")
      }
      /* result
      start in main
      in global scope main ==== start
      in global scope DefaultDispatcher-worker-1 ==== end
      end in main
      */
      ```

      ```kotlin
      public suspend fun delay(timeMillis: Long) {
         /*...*/
      }
      ```

  - block：协程的代码块

  - Job:返回一个job

    ```kotlin
    public interface Job : CoroutineContext.Element {
       /*...*/
        public fun start(): Boolean
        public fun cancel(cause: CancellationException? = null)
        public fun cancel(): Unit = cancel(null)
        public fun cancel(cause: Throwable? = null): Boolean
    		/*...*/
        public suspend fun join()
    		/*...*/
    }
    ```

    与rxjava 类似返回一个disposable一样用于控制这个协程体的引用

##### 基础使用 async

```kotlin
suspend fun main() {
    println("start in ${Thread.currentThread().name}")
    val deferred = GlobalScope.async {
        println("in global scope ${Thread.currentThread().name} ==== start")
        delay(500)
        println("in global scope ${Thread.currentThread().name} ==== end")
    }
    println("deferred is defined in ${Thread.currentThread().name}")
    deferred.await() // 这个方法需要在suspend 方法中调用
    Thread.sleep(1000)
    println("end in ${Thread.currentThread().name}")
}
/*
start in main
deferred is defined in main
in global scope DefaultDispatcher-worker-1 ==== start
in global scope DefaultDispatcher-worker-1 ==== end
end in DefaultDispatcher-worker-1
*/
```

我们来看一下`async`方法的实现

```kotlin
public fun <T> CoroutineScope.async(
    context: CoroutineContext = EmptyCoroutineContext,
    start: CoroutineStart = CoroutineStart.DEFAULT,
    block: suspend CoroutineScope.() -> T
): Deferred<T>
```

这里与`launch`相同，除了返回值不同。那么来看看Deferred的代码

```kotlin
public interface Deferred<out T> : Job {
    public suspend fun await(): T
    public val onAwait: SelectClause1<T>
    public fun getCompleted(): T
    public fun getCompletionExceptionOrNull(): Throwable?
}
```

- await：阻塞当前协程，直到协程执行完成。

- SelectClause1： select 之后再说

- getCompleted：

  ```kotlin
  suspend fun main() {
      println("start in ${Thread.currentThread().name}")
      val deferred = GlobalScope.async {
          println("in global scope ${Thread.currentThread().name} ==== start")
          delay(500)
          println("in global scope ${Thread.currentThread().name} ==== end")
      }
      println("deferred is defined in ${Thread.currentThread().name}")
      deferred.invokeOnCompletion {
          println("getCompleted in invokeOnCompletion is ${deferred.getCompleted()}")
      }
      Thread.sleep(1000)
      println("end in ${Thread.currentThread().name}")
  }
  /*
  start in main
  in global scope DefaultDispatcher-worker-1 ==== start
  deferred is defined in main
  in global scope DefaultDispatcher-worker-1 ==== end
  getCompleted in invokeOnCompletion is kotlin.Unit
  end in main
  */
  ```

- getCompletionExceptionOrNull：

  与`getCompleted`一致

async的几种用法

```kotlin
suspend fun doSomethingUsefulOne(): Int {
    delay(1000)
    return 1000
}

suspend fun doSomethingUsefulTwo(): Int {
    delay(500)
    return 500
}

suspend fun concurrentSum(): Int = coroutineScope {
    val one = async { doSomethingUsefulOne() }
    val two = async { doSomethingUsefulTwo() }
    one.await() + two.await()
}

fun main() {

    log("start in ${Thread.currentThread().name}")
    runBlocking {
        val time = measureTimeMillis {
            log("The answer is ${concurrentSum()}")
        }
        log("Completed in $time ms")
    }
    log("end in ${Thread.currentThread().name}")
}
```



##### withContext

```kotlin
fun <T> log(t: T) {
    println("[${Thread.currentThread().name}] $t")
}

fun main() {
    newSingleThreadContext("Ctx1").use { ctx1 ->
        newSingleThreadContext("Ctx2").use { ctx2 ->
            runBlocking(ctx1) {
                log("Started in ctx1")
                withContext(ctx2) {
                    log("Working in ctx2")
                }
                log("Back to ctx1")
            }
        }
    }    
}
/*
[Ctx1] Started in ctx1
[Ctx2] Working in ctx2
[Ctx1] Back to ctx1
*/
```



##### withTimeout

```kotlin
fun <T> log(t: T) {
    println("${DateTime.now().toString("YYYY-MM-DD HH:mm:ss:SSS")} >_ [${Thread.currentThread().name}] $t")
}

suspend fun main() {

    log("start in ${Thread.currentThread().name}")
    withTimeout(1000) {
        log("in global scope ${Thread.currentThread().name} ==== start")
        delay(2000)
        log("in global scope ${Thread.currentThread().name} ==== end")
    }
    Thread.sleep(1000)
    log("end in ${Thread.currentThread().name}")
}
```

```kotlin
public suspend fun <T> withTimeout(timeMillis: Long, block: suspend CoroutineScope.() -> T): T
```





##### 拦截器

```kotlin
suspend fun main() {
    GlobalScope.launch(MyContinuationInterceptor()) {
        myLog(1)
        synchronized(this) {

        }
        val job = async {
            myLog(2)
            delay(1000)
            myLog(3)
            "Hello"
        }
        myLog(4)
        val result = job.await()
        myLog("5. $result")
    }.join()
    myLog(6)
}

fun myLog(int:Int) {
    println("[${Thread.currentThread().name}] $int")
}


fun myLog(str:String) {
    println("[${Thread.currentThread().name}] $str")
}

class MyContinuationInterceptor : ContinuationInterceptor {
    override val key = ContinuationInterceptor
    override fun <T> interceptContinuation(continuation: Continuation<T>) =
        MyContinuation(continuation)
}

class MyContinuation<T>(val continuation: Continuation<T>) : Continuation<T> {
    override val context = continuation.context
    override fun resumeWith(result: Result<T>) {
        myLog("<MyContinuation> $result")
        continuation.resumeWith(result)
    }
}

```

##### 上下文

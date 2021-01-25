---
title: 'Kotlin 协程本质'
date: 2019-02-11T19:30:08+10:00
draft: false
weight: 3
summary: "kotlin协程 几问"
---

> [官方文档](https://www.kotlincn.net/docs/reference/coroutines/coroutines-guide.html)
>
> 协程是jetbrains 提供的一套 用于简化 异步编程的API。其最重要的一点就是当他挂起时，不会阻塞其他协程，协程之间切换几乎没有额外的损耗。


### kotlin 协程是什么

- kotlin 协程在jvm上的 实现就是一套线程框架，这是由于kotlin的协程中的调度器都是基于线程之心执行的

### 上下文包含的内容

- 所有的可能的元素

  ```kotlin
  public interface ThreadContextElement<S> : CoroutineContext.Element
  public interface ContinuationInterceptor : CoroutineContext.Element
  public interface CoroutineExceptionHandler : CoroutineContext.Element
  public interface Job : CoroutineContext.Element
  
  ////
  public abstract class AbstractCoroutineContextElement(public override val key: Key<*>) : Element
  public data class CoroutineName(val name: String) : AbstractCoroutineContextElement(CoroutineName)
  internal class YieldContext : AbstractCoroutineContextElement(Key)
  public abstract class CoroutineDispatcher :
  	AbstractCoroutineContextElement(ContinuationInterceptor), ContinuationInterceptor
  public object NonCancellable : AbstractCoroutineContextElement(Job), Job
  internal data class CoroutineId(    val id: Long) :
  	ThreadContextElement<String>, AbstractCoroutineContextElement(CoroutineId)
  internal class AndroidExceptionPreHandler :
      AbstractCoroutineContextElement(CoroutineExceptionHandler), CoroutineExceptionHandler
  ```

- Job

- ContinuationInterceptor 拦截器

- 错误处理

- name

- id


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



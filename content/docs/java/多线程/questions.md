---
title: '多线程--面试题'
date: 2020-02-11T19:30:08+10:00
draft: false
weight: 3
---

### synchronized 修饰普通方法和静态方法的区别？什么是可见性？
  - synchronized 锁的一定是一个具体对象，synchronzied修改的是对象头的内容，
   - synchronized 静态方法加的是 位于方法区中的class对象
   - synchronized 加在普通方法上锁的是this对象
  - 可见性：
   - 根据jmm，在线程的工作内存中修改一个变量，其他线程将看不到他在这个线程的修改，这就是可见性。
   - 线程从主内存中读取变量到工作内存中，修改运行在线程的工作内存中，不定时地将数据写回主内存。
   - 使用volatile 可以在线程修改变量之后，强行要求将其写回主内存，并且在其他协程使用的时候，强行要求其从主内存再读取一次

### 锁分哪几类？
  按照各个角度来分：
  - 线程要不要锁住资源
    - 锁住：悲观锁
    - 不锁住：乐观锁
  - 锁住同步资源失败，线程要不要阻塞？
    - 阻塞：
    - 不阻塞：自旋锁 适应性自旋锁
  - synchronized的为了优化所定义的几个级别的锁
    - 偏向锁
    - 轻量级锁
    - 重量级锁
  - 多个线程竞争的时候是否公平
    - 公平锁 排队锁
    - 非公平锁 先尝试获取一次锁，再排队
  - 一个线程中的多个流程能不能获取同一把锁
    - 可重入锁
    - 非可重入锁
  - 多个线程能不能共享一把锁
    - 共享锁 （读锁）
    - 排他锁 （写锁）


### cas 无锁编程的原理
  利用了现代cpu提供的compare and swap 原子操作能力，辅助以自旋直到完成为止等方式实现了无锁化编程的能力。

  - cas的三大问题
    - aba问题
    - 开销问题
    - 只能进行一些简单操作

### reentrantLock的实现原理
  可重入锁。显式锁的一种实现。内部实现了aqs的子类Sync。aqs内部维护了一个state，当线程获取一次锁state就加一，释放一次锁state就减一。当state为0的时候锁将被释放。

### aqs原理
  - aqs 是clh队列思想的一种变体实现。是juc的一个基础实现
  - aqs内部维护了一个state（用来表示锁的同步状态）和一个链表（线程的竞争锁的排队）。
  - 使用模版方法的设计模式，通过子类来继承实现各式各样的显式锁。子类只需实现几个模版方法即可，比如tryAcquire tryRelease等几个方法
  - 各种实现：CountDownLatch、信号量、读写锁，共享锁等
  - 每个线程通过cas 自旋，之后park来实现线程的等待


### synchronized 的原理以及于 reentrantlock的区别
  synchronized 修饰方法和放置在同步代码块上会有不同，放到方法上会添加方法的flag，而放置到同步代码块上将添加虚拟机指令 monitorenter 和monitorexit。两种方式实现是一样的。
  monitorenter都是通过修改monitor对象的markword作为标记来实现的，通过获取monitor对象的所有权来进行加锁。
  而reentrantlock是通过aqs的原理实现的显式锁，reentrantlock 提供了非公平锁和公平锁的实现，而synchronzied是非公平的加锁的

### synchronized做了哪些优化
  锁消除
  锁粗化
  逃逸分析
  https://blog.csdn.net/qq_26222859/article/details/80546917
  偏向锁
  轻量级锁
  重量级锁

### volatile 能否保证线程安全？在dcl上的作用是什么
  dcl: double check lock
  new 一个对象 不是原子操作，这里至少会有3步，1、内存分配。2、初始化内存空间。3、将引用指向内存空间
  由于指令重排序的原因可能会产生第三步在第二步之前的情况，这时候别的进程访问就可能导致npe的情况存在。
  volatile 关键字抑制指令重排序

### volatile和synchronized的区别

volatile 只保证了可见性
synchronized 可见性 排他性 内置锁的机制

### 什么是守护线程，你是如何退出一个线程的

todo 守护线程
中断的理解

stop 方法可能导致线程在不确定的情况下运行

### sleep、wait、yield的区别、wait的线程如何唤醒它

wait是继承自object的 有配套的 notify和notifyAll方法，在使用wait的时候必须通过synchronzied 关键字获取锁，在wait的时候会释放相应的锁等待别的线程notify或者notifyAll

yield：让出cpu的执行权，等待cpu的调度回来执行

sleep：休眠一定时间

锁的状态：当sleep 和yield的时候线程持有锁，不会释放锁，wait将会释放锁

### sleep 是可中断的吗
当然可以中断

### 线程生命周期

new：线程刚刚创建等待调用start启动
runnable： 调用start之后线程进入可执行状态，等待cpu调度。该状态也包括线程处于执行状态，由于这两个状态有cpu决定，所以java中混为一谈
terminated：线程run方法跑完进入结束状态
等待：wait join park 进入等待状态，等待唤醒，notify notifyAll unpark将能唤醒线程
等待超时：sleep wait join parkNanos parkUntil 进入一段时间的休眠。notify notifyAll unpark唤醒线程或者等待超时自动唤醒
阻塞：synchronized

### threadlocal是什么
threadlocal 允许每个线程保存一个线程专属的副本，

### 线程池的基本原理

为什么使用线程池
- 提高线程的可管理性
- 降低线程的开销
- 提高响应速度

线程池的几个参数的意义


### 有三个线程t1、t2、t3 怎么确保它们按照顺序执行
- join： t3的方法里面调用t2的join t2 调用t1的join

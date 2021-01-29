---
title: '多线程--synchronized'
date: 2020-02-11T19:30:08+10:00
draft: false
weight: 3
---

### synchronized(this)

synchronized 在底层字节码的实现由这两个实现

- monitorenter

拿到monitor的所有权，进入代码块

- monitorexit

每一个对象都有一个monitor与之关联。
字节码获取一个moniter对象的所有权就获取了一个锁

### public synchronized void test()


### 对象头

### 优化synchronized 的性能设计了多个锁的量级

- 对象头


- 偏向锁 （经过统计发现，现代cpu 的锁大部分都是由同一个线程反复获得，所以设计了偏向锁，连自旋都不做了）
- 轻量级锁 - 为了避免上下文切换这类重量级操作。自旋锁cas。
适应性自旋锁 自旋锁次数由虚拟机动态调整。
自旋时间不大于上下文切换所需要的时间，否则就超出了 轻量级锁的意义
- 重量级锁 上下文切换


- 当偏向锁撤销的时候需要释放锁会导致stop the world。 这就引起一个问题，

- 为什么偏向锁撤销的时候会stop the world
  因为撤销偏向锁的时候需要修改堆栈的对象的对象头内容，如果这时候不


- 偏向锁 适用于 只有一个线程访问同步块场景，因为偏向锁的撤销会导致stoptheworld。

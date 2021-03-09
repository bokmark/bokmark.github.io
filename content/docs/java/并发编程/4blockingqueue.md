---
title: '阻塞队列和线程池原理'
date: 2020-02-11T19:30:08+10:00
weight: 4
summary: "阻塞队列，线程池的实现原理"
---

# 阻塞队列 blocking queue

## jdk中的阻塞队列的实现
- ArrayBlockingQueue 数组构成的有界阻塞队列
- LinkedBlockingQueue 链表组成的有界阻塞队列
- PriorityBlockingQueue 支持优先级排序的无界阻塞队列
- DelayQueue 优先级队列实现的无界阻塞队列
- SynchronousQueue 不存储元神的阻塞队列
- LinkedTransferQueue 链表组成的无界阻塞队列
- LinkedBlockingDeque 链表组成的无界阻塞队列


# 线程池

![线程池示意图](/images/multithread.jpg)

### 线程池的接口
- exectuor
- execotorsevice
- 线程池怎么选择：cpu密集型 io密集型，混合型

### 线程池的参数意义

### 线程池的实现原理

### ThreadPoolExecutor 源码解析

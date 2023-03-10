---
title: 'JMM'
date: 2020-12-01T15:14:39+10:00
weight: 2
---

# JMM - Java Memory Model
在Java虚拟机规范中试图定义一种Java内存模型（Java Memory Model，JMM）来屏蔽各个硬件平台和操作系统的内存访问差异，以实现让Java程序在各种平台下都能达到一致的内存访问效果。<br/>
注意，为了获得较好的执行性能，Java内存模型并没有限制执行引擎使用处理器的寄存器或者高速缓存来提升指令执行速度，也没有限制编译器对指令进行重排序。也就是说，在java内存模型中，也会存在缓存一致性问题和指令重排序的问题。
Java内存模型规定所有的变量都是存在主存当中（类似于前面说的物理内存），每个线程都有自己的工作内存（类似于前面的高速缓存）。线程对变量的所有操作都必须在工作内存中进行，而不能直接对主存进行操作。并且每个线程不能访问其他线程的工作内存。

## 现代计算机原理 cpu 多级缓存模型

![cpu 多级缓存模型](/images/jmm.jpg)
线程的工作内存90% 都是以cpu中的高速内存。


- 工作内存和主内存是抽象概念

- 虚拟机中每个线程有一个工作内存，只有一个主内存
- `count++` 这个操作 在jmm中是怎么工作的
- 每个线程只允许操作自己线程的工作内存不能操作主内存和别的线程的工作内存

### jmm 导致的并发安全问题

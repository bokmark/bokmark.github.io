---
title: 'volatile'
date: 2020-02-11T19:30:08+10:00
weight: 5
---

# volatile
volatile 是什么？ 一个轻量级的同步工具，当一个共享变量被volatile修饰的时候他就具有了以下两种情况
https://www.cnblogs.com/dolphin0520/p/3920373.html
https://www.cnblogs.com/wangwudi/p/12303772.html
> jmm
### volatile 如何保证可见性

### volatile 可见性
最轻量级的同步工具，只能保证可见性，在线程的工作内存中对变量的修改会马上同步到主内存中。只能保证可见性。

### volatile 防止重排序

现代cpu 可以一次执行多个指令，从而 引入了 流水线和重排序
重排序缓存。

arm 架构拥有 3级流水线，以为这可能同时执行3条指令。

单线程情况下 重排序 将会保证执行结果相同。但是多线程情况下导致出现混乱的情况。

### volatile使用场景
一个线程写，多个线程读

### volatile 实现原理

lock 前缀 汇编指令
lock指令添加之后 线程的工作内存中使用到变量的时候必须重新从主内存中重新读取一遍
               线程的工作内存中对变量进行修改之后必须立马写会主内存

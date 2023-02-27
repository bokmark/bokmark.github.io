---
author: Bokmark Ma
title: "各种面试的答案 - 同步"
date: 2022-10-16T00:35:27+08:00
draft: true
description: 面试题 结合
slug: interview/sync
categories:
    - Interview
tags:
    - 面试
    - 同步
---

# 各种面试的答案 - 同步 

## 单例的写法，DCL，volatile的原理。
> volatile作用，防止指令重排序，保证可见性，保证单个操作的原子性。在JMM模型下，为了优化提升计算速度，语句指令是可以不按照代码的顺序执行的，处理器只需要保证结果相同即可，如果给变量添加上volatile，那么在写这个变量的时候，会强制将线程的工作内存中的内容写回主内存，并且使其他线程的工作内存中相关的部分失效。这种机制是利用了cpu提供的lock指令，这种指令将会强制将工作内存同步回主内存，而这也意味着该线程中在lock指令之前的代码都需要执行完成，这也是这种技术被称为内存屏障的缘故。
```java
public class Singleton {
   // 为什么用volatile
   private static volatile Singleton s;
   private Singleton() {
   }
   public static Singleton getInstance() {
        // 为什么要double check
        if (s == null) { // 第一遍 为了效率
            synchronized(Singleton.class) {
                if (s == null) { // 第二遍为了安全
                    s = Singleton(); // volatile 防止指令重排序。
                }
            }
      }
      return s;
   }
}
```
### 指令的重排序
什么事指令的重排序？
为了提高效率，在多cpu的情景下，处理器可能对代码进行乱序执行，并在之后将结果进行重组以确保执行结果和正序执行结果一致。而这种情况就是指令的重排序。

### 如何保证可见性
JMM中，线程有自己专属的工作线程，进行计算时，变量从主内存加载到工作内存中，在之后写回工作内存。而这种情况就导致了线程的的工作内存之间互相时不可见的。而被volatile对象修饰的

防止指令重排序，程序的代码可能进行指令的重排序，而被volatile修饰的变量在赋值之后将会进行执行一条lock的命令，也叫内存屏障，这将会能保证可见性和防止指令的重排序。而由于内存屏障，所有相关的线程在写volatile变量的时候都会写回主线程，所以在写volatile变量之前其他的计算都需要完成，这也就是内存屏障的原理。
 
### synchronized
[参考文档](https://zhuanlan.zhihu.com/p/101156763)
### Lock
[lock](https://tech.meituan.com/2019/12/05/aqs-theory-and-apply.html)


## synchronized 、lock、volatile 之间的原理与区别。
> - synchronized关键字可用于修饰方法以实例，修饰实例的时候，会在同步代码块两端生成monitorenter和monitorexit字节码指令。当执行到monitorenter的时候需要当前线程拥有对象的所有权，才可以继续执行，否则将会被挂起，等待所有权的释放。修饰方法时，区分静态方法和普通方法，静态方法相当于修饰class对象实例，修饰普通方法相当于修饰当前对象实例。每个对象实例都是由对象头，实际数据，对齐填充组成。对象头中包含了markword和类型指针，以及可能存在的数字长度。markword里面就包含了sync关键字所需要的关键信息，偏向锁，轻量锁，重量锁所需要的内容。当一个对象来到重量锁的时候，会有一个monitor对象对应。monitor对象里面包含了owner、enter set、waitset等关键信息。
> - Lock，是居于aqs实现的一套同步工具。AQS内部有一个volatile的int属性state，用于标记锁的状态。如果资源空闲， 通过cas操作将state标记为1，并且将锁的工作线程设置为当前线程。如果资源繁忙，按照是会先进行一次try的操作。之后将线程打包成node进入队列，（CLH变种双端队列）并且开始自旋不断去尝试获取锁。
> - volatile，



## 一个实例中的synchronized 是怎么锁定的。
## hashmap 和 concortenthashmap 的原理与区别
## 多线程实现的几个方法
## file lock
## thread pool的四种实现
## wait 和sleep的区别
## 什么是happend-before原则
---
author: Bokmark Ma
title: "亲自遇到过的面试题"
date: 2021-10-26T00:59:00+08:00
description: 面试题合集
slug: interview/intro
categories:
    - Interview
tags:
    - 面试
---


# 面试题目

## 美团

- 泛型
    - 泛型的意义或者作用
    - 泛型的擦除
    - kotlin 与 java 的泛型区别
        - kotlin 星投影，in out
        - java super extends
- synchronized 与lock 的 原理与区别
- volatile的原理，以及与synchronized的区别。java 单例的写法，以及为什么要加 volatile。
- UI卡顿监控
- 图片大小在内存中是如何计算的
- 性能优化 ，内存优化，如何处理
- 内存泄露，leakcanary的原理 
- hashmap 和 concortenthashmap 的原理与区别
- https 和http的区别。 https加密的过程 。ssl加密的过程
- 多线程实现的几个方法
- 算法：5x4的格子走到对头需要的步数，以及相关的步数，有没有更好的解决方案
- Handler原理，post 和 send的区别
- binder的机制原理
- hanlder idle
- tcp三次握手，四次挥手
- 进程间如何并发
- file lock
- sp的 anr在 android7之前
- sp的作用
- Activity 冷启动的过程
- instrumentation的作用
- 如何知道当前Activity的是否在前台
- 做sdk需要注意的地方
- hashmap 扩容因子
- 为什么是两倍扩容
- 为什么是8开始转红黑树
- 同步屏障在什么时候可能用到
- 二叉树查找，优化
- anr的原理

## 快手

- 一个实例中的synchronized 是怎么锁定的。
- 性能优化
- dcl的单例
- 两个链表（有共同的节点 称为相交），如何判断两个链表相交
- 两个链表合并，按顺序排序合并，写代码
- glide 的gif 实现原理、圆形图片实现原理。
- tcp 三次挥手
- tcp udp的区别
- glide 原理
- lru设计原理，双向循环链表
- bitmap如何计算在内存中的大小
- 模块话设计，分层设计

## 小米

- object 和  companion object
- 1、类加载（双亲委派）boot、path、dex
- 2、执行器：编译器、JIT、ART
- 3、对象的创建（new）：逃逸分析
- 4、GC root：
- 5、哪些对象会放到老年代
- 6、卡顿  OTA
- 7、launchMode：
    - 各类样子
    - TaskAffinity
- 知道的设计模式：桥接模式、适配器、建造者、模板、单例、装饰着、工厂、抽象工厂、策略模式、享元模式的概念理论
- Handler的原理
- 六大设计规则 solid  开闭、里氏替换原则、依赖倒置、单一职责、接口隔离、迪米特、组合复用
    单一职责原则（Single Responsibility Principle）；
    开闭原则（Open Closed Principle）；
    里氏替换原则（Liskov Substitution Principle）；
    迪米特法则（Law of Demeter），又叫“最少知道法则”；
    接口隔离原则（Interface Segregation Principle）；
    依赖倒置原则（Dependence Inversion Principle）。
- surfaceview 和 textureview的原理
- wait 和sleep的区别
- 如何判断链表中有环

## 爱奇艺

- 事件分发流程
- 生产者消费者模式手写


## 滴滴

- 事件分发流程：具体流程，那么如果父节点拦截了move事件，子节点会怎么样
- 如何让子view收不到事件节点
- 为什么用view。post 来获取宽高
- 动画的原理，属性动画如何运行起来
- 动画的分类，哪些动画
- 内存优化的具体措施
- 属性动画的原理
- kotlin 泛型和Java泛型最大的区别
- retrofit okhttp 原理 及分层
- 同步屏障的使用情况，vsync消息
- 了解fragment嘛
- aidl 修改之后会如何。
- apk 打包流程
- 广播如何实现跨进程
- 责任链模式，组成一条链，一个一个的传递，直到某个节点处理事件。
- kotin copy 是深拷贝还是浅拷贝
- activity 为什么会生命周期变化
- 为什么装饰模式contextimpl
- 内存区域
- fragment原理
- art 和 dalvik 的区别，为什么用art代替dalvik
- thread pool的四种实现
- ibinder的实现原理
- java 虚拟机和android虚拟机的区别

















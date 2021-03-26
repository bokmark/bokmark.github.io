---
title: '我的简历中涉及到的问题、知识点总结'
date: 2021-03-18
weight: 1
draft: true
tags: ["面试","简历"]
categories: ["面试","简历"]
menu: "main"
summary: "面试题合集"
---

# 简历
## 设计模式
- 享元模式 message
- 生产者消费者
- 单例
- builder
- 桥接模式
- 责任链模式
- 观察者模式
- 迭代器模式
- 适配器模式
- 模板方法模式
- 代理模式
- 装饰模式
- factory
- 抽象工厂
- 生产者消费者模式

## 数据结构


## 热修复
热修复：如何生成补丁包，开混淆之后、如何自动生成补丁包
什么时候执行热修复，怎么执行热修复，Android 版本兼容
### andfix
如何不进行类替换如何即时生效。
在native层 通过art 替换artmethod 方法。替换需要替换方法。

### robust
通过字节码插座，在编译期间，在类中加入一个static属性，在每个方法的前后都加上一个static的判断。如果这个插入的判断为空则执行源代码，否则执行热修复的代码。
有一个patch的后台线程去不断的下载patch

为什么不修改dexpathlist，而是要修改dexpathlist里面额dexdelements数组。 DexPathList mock 太麻烦。。。

### tinker
增量更新 使用bspatch bsdiff 工具， 增量更新dex文件。

什么时候修复？
越早越好，在有bug的类被修复之前。
后面的dex要删除嘛？
不能一个dex中有很多的其他的类。

怎么生成补丁包？ dx 工具可以将某个class文件生成dex文件。

bootclassloader 加载Android framework的代码
pathclassloader 加载自己写的class的加载。
dexclassloader 加载一些额外的class


### 反射的原理
反射 读取到类加载过程中在堆中生成的class对象，通过它去访问方法区中的数据。然后执行。

#### 反射为什么慢
- 反射可能反射到没加载初始化的类，从而触发加载的流程
- 反射过程中会产生一些没用用的中间类，比如参数的object数组
- 反射会导致jvm对类的优化失效，相较于正常访问会更慢


## classloader 双亲委派机制 类的生命周期
类的生命周期：加载，连接（验证，准备，解析）（比较复杂和加载、初始化交替进行），初始化，使用，卸载。
- 加载阶段：将类文件读进方法区，然后在堆中生成一个class对象，这个对象就是这个类在方法区的各种数据的入口。
- 数组类型不同
- 连接阶段：
  - 验证：校验class对象的合法性，有没有重复属性等。保证这个类是可以被jvm使用的
  - 准备：给static的属性分配内存，初始化虚拟机的默认值。
  - 解析：在解析阶段，jvm会将所有的类或接口名、字段名、方法名转换为具体的内存地址。
- 初始化： 有且仅有五种状态会进行 初始化
  - new putstatic getstatic invokestatic 遇到这几个字节码的时候
  - 反射
  - 优先将父类初始化
  - main 方法所在的类 初始化
  - 动态加载
- 使用
- 卸载
  - 没有实例
  - classloader卸载

### 双亲委派机制
- 类的唯一性
- 保证系统代码的安全
- 避免重复加载

## Android runtime
android runtime

## UI
## 事件分发

## kotlin

## 多线程
### JMM
### synchronized 原理
### aqs 原理
### cas 原理
### readwritelock用法及原理

## 垃圾回收算法
## AMS、pms、wms 原理

## 优化
### 内存优化
- 内存泄漏检查
  - handler
  - context
  - 单例
  - 匿名内部类
  - 反注册
  - 静态属性
  - 集合、数组
  - 动画
  - 资源没有关闭


- 一些大对象的酌情使用虚引用
- 主动获取当前app的内存使用情况，如果超过多少 主动释放一些资源
- oom_adj /proc/pid/oom_adj 查看
- 内存抖动：频繁gc，内存碎片化增加了oom的概率
- 内存泄漏
- 内存溢出：
  - 创建线程过多，超过1000个就会报错误
  - 堆栈内存溢出
  - 无连续内存
  - fd数量超出限制
  - 虚拟内存不足

- 通过命令分析 dumpsys meminfo
- mat
  - incoming reference、 outgoging reference
  - 深堆，浅堆
  - 找到我们自己的包里的对象，然后查看引用路径，去排查问题。

- profiler
https://developer.android.google.cn/studio/profile/memory-profiler?hl=zh-cn
Stack：您的应用中的原生堆栈和 Java 堆栈使用的内存。这通常与您的应用运行多少线程有关。

- 不断地旋转 切换Activity，不断地切换到其他Activity


- alloc count
- leakcanary

例子 carservice 多对象，list fragment，native 泄漏，动画导致Activity泄漏。


### spareMap
两个数组 keyvalue的对应
### arrayMap
两个数组 一个存放hash 一个存放keyvalue的数组

### 卡顿优化
（实例，）

## 开源框架库
### okhttp

### retrofit

### glide
#### 圆形怎么实现，gif怎么实现优化
#### lru cache 怎么使用？


### leakcanary
#### 怎么检测内存泄漏
#### 怎么dumpheap，怎么安全dumpheap
debug.dumphprofheap

### camera可能发生的问题，多线怎么处理，native crash的问题，没有关闭后续节点的问题
#### cameraActivity 泄漏的问题
#### 动画的问题

## 网络
### TCP、udp、sokect
### nio
### push通道，心跳，重连
### 网络优化

## MVVM
## MVP
## 模块化、组件化
## 组件化用了什么技术，实现原理

## jetpack
### paging3
### room
### lifecycle
### hilt
### livedata
### viewmodel

## 微信小程序

## android studio 插件
## camera底层原理
## libyuv
## eventbus
## tombstone
## eventbus
## binder
## viewstub
## viewpager
## crash breakpad
## 属性动画原理 其他动画原理
## 享元模式
## tcp ip socket nio
## 地图导航轨迹优化
## vlc rtsp

## Activity启动流程
launcher所在进程
systemserver 所在进程
目标进程

AMS 在systemserver中
ActivityThread，目标进程的主线程
ApplicationThread，目标用于和ams通信的binder
Instrumentation 监控Activity与系统交互的类

第一个阶段 launcher到ams
instrumentation.execStartActivity(this, ApplicationThread(LuancherBinder), token, ...)
activitymanager.startActivity()

第二阶段 ams内部管理
ams.startActivityasuser
Activitystarter(建造者模式的一个类)
- 检查权限，检查flag等信息
- 处理stack
- 检查目标进程如果不存在 通过socket，fork一个进程
- 等待bindiapplicationthread
- 通过applicationthread 调用目标进程的数据

- 如果目标进程存在则
- realstartactivity
- 通过clienttranstion 传递到 目标进程

第三阶段
启动Activity处理生命周期


startActivityunchecked launch flag
mStackSupervisor.resumeactivity

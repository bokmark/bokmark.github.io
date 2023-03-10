---
title: 'Binder 笔记'
date: 2018-11-28T15:14:39+10:00
weight: 2
summary: "Binder 面试问题"
---

# binder 的相关笔记


## 为什么要多进程
- 获得更多的内存 每个进程使用的内存大小通过 getprop dalivk.vm.heapsize可以 得到
- 风险隔离，每个进程不会影响别的进程

## 进程间通信 IPC 是为了什么
- A调用B的方法
- A获取B的数据
- A传递数据给B之后再获取数据
-

## linux 进程通信
- 管道
- socket
- 共享内存
- 信号量
- 等等

## binder 有什么优势（言下之义就是 和另外的进程间通信有什么优势）
- 性能：
  -- binder：数据只拷贝一次
  -- 共享内存：数据无需拷贝
  -- socket： 拷贝两次
- 易用性：
  -- binder： 基于c/s 架构，易用性较高，降低了应用开发者的使用难度，
  -- 共享内存：同步处理进程数据，比较麻烦
  -- socket： 通用接口，传输效率低，开销比较大
- 安全性：
  -- binder：实名，匿名， 每个应用分配一个uid
  -- 共享内存：依赖与上层协议，访问点开放不安全
  -- 依赖与上层协议，访问接入点开放不安全

## linux 基础 内核空间，用户空间。
内核态与用户态：

（1）当一个任务（进程）执行系统调用而陷入内核代码中执行时，称进程处于内核运行态（内核态）。此时处理器处于特权级最高的（0级）内核代码中执行。当进程处于内核态时，执行的内核代码会使用当前进程的内核栈。每个进程都有自己的内核栈。

（2）当进程在执行用户自己的代码时，则称其处于用户运行态（用户态）。此时处理器在特权级最低的（3级）用户代码中运行。当正在执行用户程序而突然被中断程序中断时，此时用户程序也可以象征性地称为处于进程的内核态。因为中断处理程序将使用当前进程的内核栈。

## binder 一次拷贝， 共享内存 无需拷贝 why
- 了解linux 基础 内核空间，用户空间。
-- https://www.cnblogs.com/sparkdev/p/8410350.html
-- https://www.cnblogs.com/Anker/p/3269106.html
-- https://blog.csdn.net/lvyibin890/article/details/82217193

#### binder 一次拷贝
- binder 数据发送方 准备好数据 内核 通过 copy_from_user() copy 到存放到一块 共享物理内存 内核也mmap到这一块物理内存
- 数据接收方， 通过mmap 将这块内存映射起来。memory mapping
- 用户空间 可以操作文件吗？ 不可以，用户空间只能操作虚拟内存
- mmap 可以将一块虚拟内存空间直接映射到一个 磁盘文件或者指定物理内存，当在用户空间修改文件内容时，就相当与直接操作物理内存或者文件。
- mmap 在驱动中怎么实现的？后续学习
#### 共享内存 无需拷贝
- 数据接受方，内核空间，数据接受方 全部mmap 到同一块区域
 ![binder 整体 示意图.jpg](https://upload-images.jianshu.io/upload_images/12975041-0197b49639ea5c37.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## binder驱动 如何启动
- linux 中 一切皆文件
- binder 驱动 也是一个文件 可以被mmap

## binder 如何跨进程，
首先数据放通过copyfromuser 拷贝到内核空间，数据接收方由于mmap 接收方就接收到这个数据了。

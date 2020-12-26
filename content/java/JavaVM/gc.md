---
title: 'Java 垃圾回收器 回收算法'
date: 2020-12-28T15:14:39+10:00
weight: 4
summary: " "
---

- 垃圾回收算法
  - 复制
  - 标记整理
  - 标记清理

- 回收具体算法
  - 单线程serial-new（复制、新生代） serial-old（标记整理）
  - 多线程 paraller scavange（复制、新生代 并行） paraller old（标记整理 并行）
  - Paraller new（复制、新生代，并行） cms（老年代，标记清理，并行、并发）

- cms
  - 先进行一次stoptheworld
  - 标记和gcroot直接相关的对象
  - 并发的进行标记
  - stoptheworld
  - 重新标记
  - 并发的进行垃圾清除

- g1
  - 化整为零


- 常量池
  - 静态常量池：编译期间确定的常量池 比如字面量
  - 动态常量池

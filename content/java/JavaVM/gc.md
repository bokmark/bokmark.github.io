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
  先进行一次stoptheword

- g1

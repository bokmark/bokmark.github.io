---
title: 'cas'
date: 2020-02-11T19:30:08+10:00
draft: false
weight: 3
summary: "compare and swap"
---

### 什么是cas
cas = compare and swap。机器指令级别实现的。原子操作，自旋实现，先campare 变量是否被修改过，如果被修改过，重新再读取campare再swap

### 什么时候用 cas 什么时候用 锁
cas 推荐在一些简单的场景，竞争不大的场景下使用，比如知识一些计数的工作等。

### cas 的原子性怎么保证
原子性由cpu保证，cas 是一条cpu的指令

### cas的问题
- aba问题
- 开销问题
- 只能保证一个共享变量的原子操作

### 只能保证一个共享变量的原子操作 如何解决
- atomic refrence


#### aba 问题怎么办
- atomic markable reference 只关心变量别改动过嘛
- atomic stamped reference 关心变量修改了几次

#### 开销问题
- cas 是由于 cpu指令 会一直自旋 到满足条件的，导致开销会比较大，如果竞争非常激烈那么开销问题是成倍增加

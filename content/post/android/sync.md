---
author: Bokmark Ma
title: "Sync 同步工具"
date: 2022-11-09T18:30:06+08:00
draft: true
description: 同步工具 个人总结
categories:
    - Android
tags:
    - Android
    - 同步工具
---

# 同步工具

## AQS

> [参考资料](https://tech.meituan.com/2019/12/05/aqs-theory-and-apply.html) [参考资料](https://mp.weixin.qq.com/s?__biz=MjM5NjQ5MTI5OA==&mid=2651749434&idx=3&sn=5ffa63ad47fe166f2f1a9f604ed10091&chksm=bd12a5778a652c61509d9e718ab086ff27ad8768586ea9b38c3dcf9e017a8e49bcae3df9bcc8&scene=38#wechat_redirect)

### 从ReentrantLock出发研究AQS的运行原理

```kotlin
val lock = ReentrantLock()
try {
    lock.lock
    // dosomething
} finally {
    lock.unlock()
}
```